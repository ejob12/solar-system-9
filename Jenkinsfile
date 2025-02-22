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
                // Set the Git user email and name
                sh "git config user.email 'ntuijunior1@gmail.com'"
                sh "git config user.name 'Big_zaza'"

                // Checkout the 'main' branch
                sh 'git checkout main'

                // Ensure the repository is up to date with the remote 'main' branch
                sh 'git fetch origin'
                sh 'git pull origin main'

                // Stage all changes and commit them
                sh 'git add -A'
                sh 'git commit -am "Updated image version for Build - $VERSION"'

                // Push the changes to the 'main' branch using the stored GitHub token
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh "git remote set-url origin https://${GITHUB_TOKEN}@github.com/Big-Zaza/gitops-argocd.git"
                    sh 'git push origin main'
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
