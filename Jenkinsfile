pipeline {
  agent any

  environment {
    NAME = "solar-system"
    IMAGE_NAME = 'bigzaza/argocd'
    IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
  }
  
  stages {
    stage('Unit Tests') {
      steps {
        echo 'Implement unit tests if applicable.'
        echo 'This stage is a sample placeholder'
      }
    }

    stage('Login to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'}
                echo 'Login successfully'
            }
        }
    
    stage('Build Docker Image')
        {
            steps
            {
                sh 'docker build -t ${IMAGE_TAG} .'
                echo "Docker image build successfully"
                sh 'docker image ls'
                echo "docker image was built successfully"
                
            }
        }

        stage('Push Docker Image')
        {
            steps
            {
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image push successfully"
            }
        }

    stage('Clone/Pull Repo') {
      steps {
        script {
          if (fileExists('gitops-argocd')) {

            echo 'Cloned repo already exists - Pulling latest changes'

            dir("gitops-argocd") {
              sh 'git pull'
            }

          } else {
            echo 'Repo does not exists - Cloning the repo'
            sh 'git clone -b main https://github.com/Big-Zaza/gitops-argocd'
          }
        }
      }
    }
    
    stage('Update Manifest') {
      steps {
        dir("gitops-argocd/jenkins-demo") {
          sh 'sed -i "s#${IMAGE_NAME}.*#${IMAGE_TAG}#g" deployment.yaml'
          sh 'cat deployment.yaml'
        }
      }
    }

    
    stage('Commit & Push') {
      steps {
        dir("gitops-argocd/jenkins-demo") {
            script {
                // Set the Git user email and name for the current repository (better than global settings)
                sh "git config user.email 'ntuijunior1@gmail.com'"
                sh "git config user.name 'Big_zaza'"

                // Ensure the repository is up to date with the remote 'main' branch before committing
                sh 'git fetch origin'
                sh 'git checkout main'
                sh 'git pull origin main'

                // Stage all changes
                sh 'git add -A'

                // Check if there are any changes before committing
                def commitMessage = "Updated image version for Build - ${env.VERSION}"
                def commitStatus = sh(script: "git diff --cached --quiet || echo 'changes'", returnStdout: true).trim()

                // Commit the changes if there are any staged changes
                if (commitStatus == 'changes') {
                    sh "git commit -m '${commitMessage}'"
                    sh 'git push origin main'
                } else {
                    echo "No changes to commit"
                }
            }
        }
    }
}

   
    
    stage('Successfully deployed to ArgoCD') {
      steps {
        echo "Application has been deployed to ArgoCD"
      }
    } 
  }
}
