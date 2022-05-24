pipeline {
    agent none
    parameters
    {
     string(name: "APICIP", defaultValue: '10.106.236.57', description: "APIC IP")
     string(name: "APICUSER", defaultValue: 'saukotha', description: "APIC User Name")
    }
    environment {
        WEBEXAUTH = credentials('webex-auth-token')
        WEBEXROOM = credentials('webex-roomid')
        APICHOST = "$params.APICIP"
        APICUSER = "$params.APICUSER"
        APICPASSWORD = credentials('apic_password')
   }
    stages {
        stage('Git Checkout') {
            agent {
                label 'ciscolive'
            }
            steps {
            git branch: 'main', url: 'https://github.com/saurabhkothari/jenkins-ansible.git'
            stash includes: '*' , name: 'my-ansible-playbook'
            }
        }
        stage('Get Firmware Details'){
            agent {
                label 'ciscolive'
            }
            steps {
                unstash 'my-ansible-playbook'
             
                ansiColor('xterm') {
                
                            ansiblePlaybook(
                             playbook: 'get_fw.yaml' ,
                             inventory: 'inventory',
                             extraVars: [
                                        "apic_host": "${APICHOST}",
                                        "apic_user": "${APICUSER}",
                                        "apic_password": "${APICPASSWORD}",
                                        ],
                             colorized: true)
               
                            } 
                        
                    }
                }         
    }
   post { 
        success { 
            node("ciscolive") {
            wrap([$class: 'BuildUser']) {
            sh """
                    curl --location --request POST 'https://webexapis.com/v1/messages' \
--header "Authorization: Bearer ${WEBEXAUTH}" \
--header 'Content-Type: application/json' \
--data-raw '{
  "roomId": "${WEBEXROOM}",
  
  "markdown": "Started at: ${BUILD_TIMESTAMP}\\nAuthor:${env.BUILD_USER}(${env.BUILD_USER_EMAIL})\\nJob URL: ${Job_URL}\\nBuild ID: ${BUILD_NUMBER}\\nBuild Logs: ${BUILD_URL}consoleText\\nResult: ${currentBuild.currentResult}" 
}'
         """
        }
        }
        }
        failure { 
            node("ciscolive") {
            sh """
                    curl --location --request POST 'https://webexapis.com/v1/messages' \
--header "Authorization: Bearer ${WEBEXAUTH}" \
--header 'Content-Type: application/json' \
--data-raw '{
  "roomId": "${WEBEXROOM}",
  
  "markdown": "Started at: ${BUILD_TIMESTAMP}\\nAuthor:${env.BUILD_USER}(${env.BUILD_USER_EMAIL})\\nJob URL: ${Job_URL}\\nBuild ID: ${BUILD_NUMBER}\\nBuild Logs: ${BUILD_URL}consoleText\\nResult: ${currentBuild.currentResult}" 
}'
         """
        } }
    }
}
