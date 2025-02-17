pipeline {
    environment {
        imagename = "davidinane/laravel8cd"
        dockerImage = ""
    }
    
    agent any
    stages {
        stage("Build") {
            environment {
                DB_HOST = credentials("laravel-host")
                DB_DATABASE = credentials("laravel-database")
                DB_USERNAME = credentials("laravel-user")
                DB_PASSWORD = credentials("laravel-password")
            }
            steps {
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
            steps {
                sh 'php artisan test'
            }
        }
        stage("Code coverage") {
            steps {
                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"
            }
        }
        stage("Static code analysis larastan") {
            steps {
                sh "vendor/bin/phpstan analyse --memory-limit=2G"
            }
        }
        stage("Static code analysis phpcs") {
            steps {
                sh "vendor/bin/phpcs"
            }
        }
        stage("Docker build") {
            steps {
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
            
        //stage('Push image') {
        //    steps {
        //        script {
        //            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials'){
        //            dockerImage.push("${env.BUILD_NUMBER}")
        //            dockerImage.push("latest")
        //            }
        //         }
        //    }
        //}
        
        stage("Deploy to staging") {
            steps {
                sh "sudo docker-compose up -d"
                //sh 'php artisan migrate'
            }
        }
    }
}
