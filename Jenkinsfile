def scan_type

def target
    
pipeline {
     agent any
     parameters {
         
         string defaultValue: "http://demo.testfire.net",
                 description: 'Target URL to scan',
                 name: 'TARGET'
 
         booleanParam defaultValue: true,
                 description: 'Parameter to know if wanna generate report.',
                 name: 'GENERATE_REPORT'
     }
     stages {
         stage('Pipeline Info') {
                 steps {
                     script {
                         echo "<--Parameter Initialization-->"
                         echo """
                         The current parameters are:
                             Scan Type: ${params.SCAN_TYPE}
                             Target: ${params.TARGET}
                             Generate report: ${params.GENERATE_REPORT}
                         """
                     }
                 }
         }
 
         stage('Setting up OWASP ZAP docker container') {
             steps {
                 script {
                         echo "Pulling up last OWASP ZAP container --> Start"
                         sh 'sudo docker pull owasp/zap2docker-stable'
                         echo "Pulling up last VMS container --> End"
                         echo "Starting container --> Start"
                         sh """
                         sudo docker run -dt --name owasp \
                         owasp/zap2docker-stable \
                         /bin/bash
                         """
                 }
             }
         }
 
 
         stage('Prepare wrk directory') {
             when {
                         environment name : 'GENERATE_REPORT', value: 'true'
             }
             steps {
                 script {
                         sh """
                             sudo docker exec owasp \
                             mkdir /zap/wrk
                         """
                     }
                 }
         }
 
 
         stage('Scanning target on owasp container') {
             steps {
                 script {
                     scan_type = "${params.SCAN_TYPE}"
                     echo "----> scan_type: $scan_type"
                     target = "${params.TARGET}"
                     if(scan_type == "Baseline"){
                         sh """
                             sudo docker exec owasp \
                             zap-baseline.py \
                             -t $target \
                             -x report.xml \
                             -I
                         """
                     }
                     //-x report-$(date +%d-%b-%Y).xml
                     else{
                         echo "Something went wrong..."
                     }
                 }
             }
         }
         stage('Copy Report to Workspace'){
             steps {
                 script {
                     sh '''
                         sudo docker cp owasp:/zap/wrk/report.xml ${WORKSPACE}/report.xml
                     '''
                 }
             }
         }
     }
     post {
             always {
                 echo "Removing container and images"
                 sh '''
                     sudo docker stop owasp
                     sudo docker rm owasp
                     sudo docker rmi owasp/zap2docker-stable 
                 '''
             }
         }
}
