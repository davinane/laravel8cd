pipeline {
    agent any
    stages {
        stage("Build") {
            environment {
                DB_HOST = credentials("laravel-host")
                DB_DATABASE = credentials("laravel-database")
                DB_USERNAME = credentials("laravel-user")
                DB_PASSWORD = credentials("laravel-password")
            }
            step {
                sh 'php --version'
                sh 'composer install'
                sh 'composer --version'
                sh 'cp .env.example .env'
                sh 'echo DB_HOST=${DB_HOST} >> .env'
                sh 'echo DB_USERNAME=${DB_USERNAME} >> .env'
                sh 'echo DB_DATABASE=${DB_DATABASE} >> .env'
                sh 'echo DB_PASSWORD=${DB_PASSWORD} >> .env'
                sh 'php artisan key:generate'
                sh 'cp .env .env.testing'
            }
        }
        stage("Unit test") {
            step {
                sh 'php artisan test'
            }
        }
        stage("Code coverage") {
            app.inside {
                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"
            }
        }
        stage("Static code analysis larastan") {
            step {
                sh "vendor/bin/phpstan analyse --memory-limit=2G"
            }
        }
        stage("Static code analysis phpcs") {
            step {
                sh "vendor/bin/phpcs"
            }
        }
        stage("Docker build") {
            step {
                sh "sudo docker rmi icentra/laravel8cd" 
            }
            app = docker.build("icentra/laravel8cd")
        }
            
        stage('Push image') {
            step {
                withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                    push("${env.BUILD_NUMBER}")
                    push("latest")
                }
            }
        }
        
        stage("Deploy to staging") {
            app.inside {
                sh "sudo docker-compose up -d"
                sh 'php artisan migrate'
            }
        }
        stage("Acceptance test curl") {
            step {
                sleep 20
                sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
            }
        }
        stage("Acceptance test codeception") {
            step {
                sh "vendor/bin/codecept run"
            }
            post {
                always {
                    sh "sudo docker stop laravel8cd"
                }
            }
        }
}
}
