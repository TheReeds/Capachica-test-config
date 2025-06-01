pipeline {
    agent {
        docker {
            image 'php:8.2-cli'
            args '-u root'
        }
    }

    environment {
        COMPOSER_ALLOW_SUPERUSER = "1"
    }

    stages {
        stage('Clone') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    deleteDir()
                    git branch: 'main',
                        credentialsId: 'github-token',
                        url: 'https://github.com/TheReeds/capachica-project.git'
                }
            }
        }

        stage('Prepare Environment') {
            steps {
                sh '''
                    apt-get update && apt-get install -y \
                        unzip git curl zip libzip-dev libpq-dev \
                        libcurl4-openssl-dev libonig-dev libxml2-dev

                    docker-php-ext-install mbstring curl pdo_mysql pdo_pgsql zip xml

                    curl -sS https://getcomposer.org/installer | php
                    mv composer.phar /usr/local/bin/composer

                    php -r "copy('.env.example', '.env');"
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('turismo-backend') {
                    timeout(time: 15, unit: 'MINUTES') {
                        sh '''
                            composer install
                            php artisan key:generate
                        '''
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('turismo-backend') {
                    timeout(time: 6, unit: 'MINUTES') {
                        sh '''
                            mkdir -p coverage
                            php artisan test --coverage --coverage-clover=coverage/clover.xml --coverage-html=coverage/html --log-junit=coverage/junit.xml
                        '''
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('turismo-backend') {
                    timeout(time: 5, unit: 'MINUTES') {
                        withSonarQubeEnv('sonarqube') {
                            sh """
                                sonar-scanner \
                                  -Dsonar.projectKey=turismo-backend \
                                  -Dsonar.projectName=turismo-backend \
                                  -Dsonar.sources=app \
                                  -Dsonar.tests=tests \
                                  -Dsonar.test.inclusions=**/*Test.php \
                                  -Dsonar.exclusions=**/vendor/**,**/storage/**,**/bootstrap/**,**/*.blade.php,**/database/migrations/**,**/config/** \
                                  -Dsonar.php.coverage.reportPaths=coverage/clover.xml \
                                  -Dsonar.php.tests.reportPath=coverage/junit.xml \
                                  -Dsonar.language=php \
                                  -Dsonar.sourceEncoding=UTF-8 \
                                  -Dsonar.php.file.suffixes=php \
                                  -Dsonar.duplicatedBlocks.minimumTokens=50 \
                                  -Dsonar.coverage.exclusions=**/tests/**,**/database/**,**/config/** \
                                  -Dsonar.cpd.exclusions=**/tests/**,**/database/migrations/**
                            """
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 4, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
