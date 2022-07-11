def stopPipeline = false

pipeline {
  environment {
    registry = "nikolaikunai/wordpress-sa"
    registryCredential = 'dockerhub'
  }
  agent { node { label 'node1' } }
  stages {
    stage('Clone repo') {
      steps {
        deleteDir()
        git url: 'https://github.com/nikolaikunai/sa_project_wp.git', branch: 'main'
        sh "chmod +x docker-entrypoint.sh"
      }
	  }
    stage('Building image') {
      steps {
        script {
          catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
            try {
              dockerImage = docker.build registry + ":$BUILD_NUMBER"
            }
            catch (Exception err) {
              currentBuild.result = "FAILURE"
              stopPipeline = true
              println "stopPipeline = ${stopPipeline}"
              sh "exit 1"
            }
          }            
        }   
      } 
    }
    stage('Deploy Image') {
      when {
        expression {
        !stopPipeline
        }
      }
      steps {
        script {
          docker.withRegistry( '', registryCredential ) {
          dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps {
        sh "docker rmi $registry:$BUILD_NUMBER"
        deleteDir()
      }
    }
    stage('Trigger ManifestUpdate') {
      when {
        expression {
        !stopPipeline
        }
      }
      steps {
        script {
              echo "triggering update manifest job"
              build job: 'Update_Manifests', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)], wait: false 
        }
      }
    }
  }
  post {
    success {
      slackSend color: '#2EB67D', 
                message: "Docker image creation completed SUCCESSFULLY: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
      }
    failure {
      slackSend color: '#E01E5A', 
                message: "Docker image creation completed with ERROR: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
    }
  }  
}  