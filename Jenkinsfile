pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds() // no two bulids run at a same time 
        ansiColor('xterm')
    }
    environment{
        def appVersion = '' // global var declaration 
        nexusUrl = 'nexus.bhavya.store:8081'
    }
    stages {
        stage('read the version') {
            steps {
               script {               //grovvy sysntax
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version : $appVersion"
               }
            }
        }
        stage('Build'){
            steps{
                sh """
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip 
                ls -ltr
                """
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "frontend" ,
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage('Deploy'){   
            // when{
            //     expression{
            //         params.deploy
            //     }
            // }
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'frontend CD', parameters: params, wait: false // false here indicates just the CI is success and doesn't care about CD success or failure $ bulid job indicates the node name 
                }
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir() //removes the workspace repeatdely in order to avoid the desprencies 
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}