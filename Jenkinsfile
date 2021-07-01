node {
    def app
        stage("Build") {
            app.environment {
                DB_HOST = credentials("laravel-host")
                DB_DATABASE = credentials("laravel-database")
                DB_USERNAME = credentials("laravel-user")
                DB_PASSWORD = credentials("laravel-password")
            }
            app.inside {
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
            app.inside {
                sh 'php artisan test'
            }
        }
        stage("Code coverage") {
            app.inside {
                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"
            }
        }
        stage("Static code analysis larastan") {
            app.inside {
                sh "vendor/bin/phpstan analyse --memory-limit=2G"
            }
        }
        stage("Static code analysis phpcs") {
            app.inside {
                sh "vendor/bin/phpcs"
            }
        }
        stage("Docker build") {
            app.inside {
                sh "sudo docker rmi icentra/laravel8cd"
                sh "sudo docker build -t icentra/laravel8cd ."
            }
        }
            
        stage('Push image') {
            app.inside {
                docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
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
            app.inside {
                sleep 20
                sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
            }
        }
        stage("Acceptance test codeception") {
            app.inside {
                sh "vendor/bin/codecept run"
            }
            app.post {
                always {
                    sh "sudo docker stop laravel8cd"
                }
            }
        }
}
