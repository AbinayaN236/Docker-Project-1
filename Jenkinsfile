pipeline {
    agent any
    
    environment {
        registry = "529088272063.dkr.ecr.us-west-2.amazonaws.com/docker-project-repo"
        python_image_tag = "python-image"
        sql_image_tag = "sql-image"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/sakshara-github/vanakkam-world.git'
            }
        }

        stage('Building Python image') {
            steps {
                script {
                    // Build Python Docker image
                    dockerImage = docker.build("${registry}:${python_image_tag}", "-f Dockerfile.python .")
                }
            }
        }

        stage('Building SQL image') {
            steps {
                script {
                    // Build SQL Docker image
                    dockerImage = docker.build("${registry}:${sql_image_tag}", "-f Dockerfile.sql .")
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-docker', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        // Log in to ECR
                        sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 529088272063.dkr.ecr.us-west-2.amazonaws.com'
                        
                        // Push both Python and SQL images to ECR
                        sh 'docker push 529088272063.dkr.ecr.us-west-2.amazonaws.com/docker-project-repo:python-image'
                        sh 'docker push 529088272063.dkr.ecr.us-west-2.amazonaws.com/docker-project-repo:sql-image'
                    }
                }
            }
        }

        stage('Stop previous containers') {
            steps {
                sh 'docker ps -f name=Container -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -fname=Container -q | xargs -r docker container rm'
            }
        }

        stage('Docker Run Python') {
            steps {
                script {
                    // Run the Python container
                    sh 'docker run -d -p 9096:8080 --rm --name mypythonContainer 529088272063.dkr.ecr.us-west-2.amazonaws.com/docker-project-repo:python-image'
                }
            }
        }

        stage('Docker Run SQL') {
            steps {
                script {
                    // Run the SQL container
                    sh 'docker run -d -p 9097:8080 --rm --name mysqlContainer 529088272063.dkr.ecr.us-west-2.amazonaws.com/docker-project-repo:sql-image'
                }
            }
        }
    }
}
