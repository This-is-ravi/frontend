pipeline {
    agent{
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds() 
        ansiColor('xterm')
    }
    environment{
        def appVersion = '' //Global variable declaration
        nexusUrl = 'nexus.csvdaws78s.online:8081'
        
    }
    
   
    stages {
        stage('read the version'){
            steps{
                script{  //its a Groovy script
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
                
            }

        }
       

        stage('Build'){
            steps{ // excluding the frontend.zip 
                sh """
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip 
                ls -ltr 
                """ // ls -ltr to know zip file created or not, -q removes unnessary logs in o/p
            }
        }

        stage('Nexus Artifact Upload'){ //using this code artifact wil upload to nexus
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}", // use double qouts wen usng variables
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth', // wen pushing we need cred,jenkins wil go n check auth
                        artifacts: [
                            [artifactId: "frontend" ,
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',   // frontend-1.1.0.zip
                            type: 'zip']       // java means- jar file
                        ]
                    )
                }
            }
        }

        stage('Deploy'){
            
            steps{
                script{   // we r passing the params i.e CI to trigger the CD-job
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")  //1.2.0
                    ]
                    build job: 'frontend-deploy', parameters: params, wait: false //if True, it waits till downstream CD-job completes
                }
            }
        }
    }

    
    post { 
        always {  // delete the workspace build after new build starts
            echo 'I will always say Hello again!'  
            deleteDir()
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }


}