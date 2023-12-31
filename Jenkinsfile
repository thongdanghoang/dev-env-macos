pipeline {

    agent any

    tools {
        maven 'mvn-3.9.6'
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t thongdh3401/demo-jenkins .'
                    sh 'docker push thongdh3401/demo-jenkins'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop thongdh-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm thongdh-mysql-data || echo "no volume"'

                sh "docker run --name thongdh-mysql --rm --network dev -v thongdh-mysql-data:/var/lib/mysql \
                -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i thongdh-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull thongdh3401/demo-jenkins'
                sh 'docker container stop thongdh-demo-jenkins || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name thongdh-demo-jenkins -p 8081:8080 --network dev thongdh-demo-jenkins'
            }
        }

    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}