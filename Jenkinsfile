pipeline {
    agent any

    environment {
        COMPOSER_ALLOW_SUPERUSER = "1"
        SONAR_HOST_URL = 'http://docker.sonar:9000'
        SONAR_TOKEN = 'squ_f8db1b0d99540505f8c71a9ee7d39b663e75e6d9'
    }

    tools {
        // Si Jenkins tiene PHP o Composer como herramientas configuradas, puedes agregarlas aquí
        php "PHP_8"
        composer "Composer"
    }

    stages {
        stage('Clone') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    git branch: 'main', credentialsId: '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', url: 'https://github.com/TheReeds/capachica-project.git'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('turismo-backend') {
                    timeout(time: 5, unit: 'MINUTES') {
                        sh '''
                            composer install
                            cp .env.example .env
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
                            sh '''
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
                                  -Dsonar.cpd.exclusions=**/tests/**,**/database/migrations/** \
                                  -Dsonar.host.url=$SONAR_HOST_URL \
                                  -Dsonar.login=$SONAR_TOKEN
                            '''
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
