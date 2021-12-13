pipeline {
  agent any
  environment {
    WORKSPACE = "${env.WORKSPACE}"
  }
  tools {
    maven 'localMaven'
    jdk 'localJdk'
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
      post {
        success {
          echo ' now Archiving '
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }
    stage('SonarQube Scan') {
      steps {
        sh """mvn sonar:sonar \
               -Dsonar.host.url=http://3.96.147.122:9000 \
               -Dsonar.login=0ef3908c54fa4017e8396c3b3ded1a71405a1362"""
      }
    }
    stage('Upload to Artifactory') {
      steps {
        sh "mvn clean deploy -DskipTests"
      }

    }
    stage ('Server'){
            steps {
               rtServer (
                 id: "Artifactory",
                 url: 'http://localhost:8082/artifactory',
                 username: 'ravish',
                  password: 'YouPasswordHere',
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }
    stage('Upload'){
            steps{
                rtUpload (
                 serverId:"Artifactory" ,
                  spec: '''{
                   "files": [
                      {
                      "pattern": "*.war",
                      "target": "logic-ops-lab-libs-snapshot-local"
                      }
                            ]
                           }''',
                        )
            }
        }
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }

    }
    stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to PROD') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
  }

}
