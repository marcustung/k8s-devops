def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, serviceAccount: 'jenkins', containers: [
  containerTemplate(name: 'netcore22', image: 'microsoft/dotnet:2.2.100-sdk-alpine', ttyEnabled: true, resourceRequestCpu: '200m', resourceLimitCpu: '200m', resourceRequestMemory: '400Mi', resourceLimitMemory: '400Mi'),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true, resourceRequestCpu: '200m', resourceLimitCpu: '200m', resourceRequestMemory: '400Mi', resourceLimitMemory: '400Mi'),  
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true, privileged: false, resourceRequestCpu: '100m', resourceLimitCpu: '100m', resourceRequestMemory: '200Mi', resourceLimitMemory: '200Mi')  
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  hostPathVolume(mountPath: '/home/jenkins/.nuget/packages', hostPath: '/home/.nuget/packages/')
]){
    node(label) {
        parameters {
            string(name: 'BUILD_DOCKER_IMAGE', defaultValue: 'false', description: 'Do you want to build docker image?')                    
        }

        def myRepo = checkout scm
        def gitCommit = myRepo.GIT_COMMIT
        def gitBranch = myRepo.GIT_BRANCH
        def shortGitCommit = "v-${gitCommit[0..6]}"

        try{
            githubNotify description: 'This build is good',  status: 'SUCCESS'
            
            stage('Build') {
                container('netcore22') {
                    sh """
                        dotnet restore
                        dotnet build k8s-devops.sln --no-restore -nowarn:msb3202,nu1503
                    """
                }
            }

            stage('Run unittest') {             
                println "Comming soon!"
            }

            if(params.BUILD_DOCKER_IMAGE == 'true' ) {
                stage('Buid docker image') {
                    container('docker') {
                        sh """
                            docker --version
                            echo $shortGitCommit
                            echo $REGISTRY_URL
                        """
                    }
                }
            }

            
        }
        catch(e) {
            githubNotify description: 'Err: Incremental Build failed with Error: ' + e.toString(),  status: 'FAILURE'
            throw e
        }
    }    
}