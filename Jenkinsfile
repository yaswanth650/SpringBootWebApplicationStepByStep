pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/yaswanth650/SpringBootWebApplicationStepByStep.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
     stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/cehkunal/webapp/master/owasp-dependency-check.sh"'
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
       }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package -DskipTests'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.jar ubuntu@13.127.121.234:/prod/apache-tomcat-9.0.65/lib/springbootfirstwebapplication.jar'
              }      
           }
     }
	
     stage ('DAST') {
       steps {
          sshagent(['zap']) {
            sh 'ssh -o  StrictHostKeyChecking=no ubuntu@43.205.229.246 "docker run -t owasp/zap2docker-stable zap-baseline.py -t https://13.127.121.234:8443/springbootfirstapplication/" || true'
        }
      }
    }
  }
}

