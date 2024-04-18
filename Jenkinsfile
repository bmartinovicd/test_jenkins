def send_email(job_name, job_ID, test_results) {
    echo (message: "Send email: ${job_name}, ${job_ID}, TEST RESULTS: ${test_results}")
}


pipeline {
    agent any
    parameters {
        string defaultValue: 'pipeline', name: 'GIT_BRANCH'
        booleanParam defaultValue: true, name: 'RUN_TEST'
        booleanParam defaultValue: true, name: 'SEND_EMAIL'
    }
    
    
    stages {
        stage('Download') {
            steps {
                cleanWs()
                echo (message: "Download")
                dir('pipeline') {
                    git (
                        branch: params.GIT_BRANCH,
                        url:'https://github.com/KLevon/jenkins-course'
                    )
                }
                rtDownload(
                    serverId: 'Artifactory',
                    spec: '''{
                        "files": [
                        {
                            "pattern": "generic-local/printer.zip",
                            "target" :  "pipeline_folder_zip/"
                        }
                        ]
                    }'''
                )
                unzip (
                    zipFile: "pipeline_folder_zip/printer.zip",
                    dir: "pipeline"
                )
            }
        }
        stage('Build') {
            steps {
                echo (message: "Build")
                dir('pipeline') {
                    bat (
                        script: 'Makefile.bat'
                    )
                }
                withCredentials (
                    [usernamePassword(credentialsId: 'bojana_user', passwordVariable: 'psw', usernameVariable: 'usr')]
                ) {
                    echo "USERNAME: $usr"
                    echo "PASSWORD: $psw"
                    echo "bojanam"
                }
            }
        }
        stage('Tests') {
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps {
                echo (message: "Tests")
                script {
                    def modules = ['printer', 'scanner', 'main']
                    env.output = ""
                    for (element in modules) {
                        dir('pipeline') {
                            env.output += bat ( script: """Tests.bat ${element}""" , returnStdout: true).trim()
                        }
                    }
                }
            }
        }
		stage('Dynamic') {
            steps {
                echo (message: "DYNAMIC")
                
            }
        }
        stage('Publish') {
            steps {
                echo (message: "Publish")
                script {
                    zip (
                        zipFile: "zip_file.zip",
                        archive: true,
                        dir: "pipeline"
                    )
                    bat ("exit 1")
                }
                rtUpload(
                    serverId: 'Artifactory',
                    spec: """{
                        "files": [
                        {
                            "pattern": "zip_file.zip",
                            "target" :  "generic-local/release/${env.BUILD_ID}"
                        }
                        ]
                    }"""
                )
            }
        }
        
    }
    post {
        failure {
            script {
                if(params.SEND_EMAIL) {
                    send_email(env.JOB_NAME, env.BUILD_ID, env.output)
                }
            }
        }
    }
}