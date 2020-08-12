pipeline {
    agent none

    options {
      buildDiscarder(logRotator(numToKeepStr: '5'))
    }    

    stages {
        stage('Checkout'){
            agent {
                docker {
                    image 'ruby:2.6.3'
                    args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker'
                }
            }
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Code Backup'){
            agent any
            when{
                branch 'master'
            }
            steps {
                script {
                    DATE_TAG = java.time.LocalDate.now()
                    DATETIME_TAG = java.time.LocalDateTime.now()
                }
                sh "echo ${JOB_BASE_NAME}"
                sh "echo ${JOB_NAME}"
                sh "sleep 10"
                sh "tar cvzf ${JOB_BASE_NAME}-${DATE_TAG}.tar.gz -C $WORKSPACE/ ."
                sh "mv $WORKSPACE/${JOB_BASE_NAME}-${DATE_TAG}.tar.gz /root/code_backup/yatra-live-service/"
            }
        }
        stage ('Starting Down Stream Job') {
            when{
                branch 'master'
            }
            steps {
                build job: 'yantra/yantra-deployment', \
                parameters: [string(name: 'Host_Name', value: "yantra-live-service"), \
                string(name: 'Project_Type', value: "yantra-live-service"), \
                string(name: 'Build_Name', value: "${JOB_BASE_NAME}-${DATE_TAG}.tar.gz")], \
                propagate: false
            }
        }
    }    
}