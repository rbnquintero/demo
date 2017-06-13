node {
    // Setup the Docker Registry (Docker Hub) + Credentials 
    registry_url = "https://index.docker.io/v1/" // Docker Hub
    docker_creds_id = "docker" // name of the Jenkins Credentials ID
    build_tag = "testing" // default tag to push for to the registry
    
    sh "groups"
    sh "echo $PATH"
    sh "docker --version"
    
    stage 'Building Container for Docker Hub'
    docker.withRegistry("${registry_url}", "${docker_creds_id}") {
        stage 'Checking out GitHub Repo'
        git url: 'https://github.com/rbnquintero/demo.git', credentialsId: 'github'
    
        sh "git rev-parse HEAD > .git/commit-id"
        def commit_id = readFile('.git/commit-id').trim()
        println commit_id
    
        stage "gradle build"
        sh "./gradlew buildDocker -Ppush -Pversion=${BUILD_NUMBER}"
    }
    
    stage "kubernetes deployment"
    def nodeip = deployContainer("rbnquintero/demo:${BUILD_NUMBER}","alone");
    echo '${nodeip}'
}

def deployContainer(image, env) {
    docker.image('lachlanevenson/k8s-kubectl:v1.5.2').inside {
        withCredentials([[$class: "FileBinding", credentialsId: 'KubeConfig', variable: 'KUBE_CONFIG']]) {
            def kubectl = "kubectl  --kubeconfig=\$KUBE_CONFIG --context=${env}"
            def deploy = "kube-demo-deployment"
            
            sh "${kubectl} set image deployment/${deploy} ${deploy}=${image}"
            sh "${kubectl} rollout status deployment/${deploy}"
            
            def ip="${kubectl} get service/${deploy} -o jsonpath='{.items[0].status.addresses[0].address}'"
            def port="${kubectl} get svc kube-demo-deployment -o jsonpath='{.spec.ports[0].nodePort}'"
            return 'http://'+ip+':'+port
        }
    }
}