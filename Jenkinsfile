def sendEmail(arg1, arg2, arg3, arg4){
    echo (message: arg1 + " " +arg2 + " "+ arg3
     + " " + arg4
     )
}

def output = ""

pipeline {
    agent any
    
    parameters {
        string defaultValue: 'katarina', name: 'ARTIFACT_NAME'
        booleanParam defaultValue: false, name: 'FAIL_PIPELINE'
        booleanParam name:'RUN_TEST'
        
    }
    
    
    
    stages {
        stage('Download') {
            steps {
                cleanWs()
                echo (message: "Dowmload stage")
                dir ('pipeline'){
                    git (
                        branch: 'pipeline',
                        url: 'https://github.com/KLevon/jenkins-course'
                        )
                }
                rtDownload (
                    serverId: 'Artifactory',
                    spec: '''{
                          "files": [
                            {
                              "pattern" : "generic-local/libraries/printer.zip",
                              "target": "./printer.zip"
                            }
                            ]
                            }'''
                                    
                            )
                unzip (
                    zipFile: "libraries/printer.zip",
                    dir: "pipeline/")
                    
                
            }
        }
        stage('Build') {
            steps {
                echo (message: "Build stage")
                
                withCredentials (
                    [usernamePassword(credentialsId: '1234', passwordVariable: 'psw', usernameVariable: 'usr')]
                    ){
                        echo (message: "$psw $usr")
                    }
                
                bat (
                    script: """
                    cd pipeline
                    Makefile.bat
                    """
                    )
            }
        }
        
        
        
        stage('Publish') {
            steps {
                echo (message: "Publish stage")
                
                script {
                    zip ( 
                        zipFile: "pipeline.zip",
                        archive: true,
                        dir: "pipeline"
                        )
                }
                
                rtUpload (
                    serverId: 'Artifactory',
                    spec: """{
                      "files": [
                        {
                          "pattern": "pipeline.zip",
                          "target": "generic-local/release/katarina/${env.BUILD_ID}/${params.ARTIFACT_NAME}.zip"
                        }
                        ]
                        }"""
                    )
                 
                
            }
        }

        stage ('Dynamic'){
            when {
              branch 'feature/multi/*'
            }
            steps{

            echo (message:  "Dynamic stage")}
        }
        
        stage('Tests') {
            when{
                equals expected: true,
                actual: RUN_TEST
            }
            
            steps{
                script{
                    def array = ["printer", "scanner", "main"]
                    for (element in array){
                       output  += 
                       bat(
                           script: """
                           cd pipeline
                           Tests.bat $element
                           """
                           , returnStdout: true
                           ).trim()
                    }
                    
                    if ( params.FAIL_PIPELINE == true) {
                        bat( 
                            script: """
                            exit 1
                            """
                            )
                    }
                    
                }
               
                
            }
        }
        
     }
     
     post{
        failure{
            
                script{
                    sendEmail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, output)
                }
            
            }
    }
    }
    
    
    
