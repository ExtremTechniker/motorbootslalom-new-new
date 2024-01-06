pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from repository
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                // Install Composer dependencies
                sh 'composer install'
            }
        }
        stage('Static Analysis') {
            parallel {
                stage('CodeSniffer') {
                    steps {
                        sh 'vendor/bin/phpcs --standard=phpcs.xml .'
                    }
                }
                stage('PHP Compatibility Checks') {
                    steps {
                        sh 'vendor/bin/phpcs --standard=phpcs-compatibility.xml .'
                    }
                }
                stage('PHPStan') {
                    steps {
                        sh 'vendor/bin/phpstan analyse --error-format=checkstyle --no-progress -n . > build/logs/phpstan.checkstyle.xml'
                    }
                }
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'vendor/bin/phpunit'
                xunit([
            thresholds: [
                failed(failureThreshold: '0'),
                skipped(unstableThreshold: '0')
            ],
            tools: [
                PHPUnit(pattern: 'build/logs/junit.xml', stopProcessingIfError: true, failIfNotNew: true)
            ]
        ])
                publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: 'build/coverage',
            reportFiles: 'index.html',
            reportName: 'Coverage Report (HTML)',
            reportTitles: ''
        ])
                publishCoverage adapters: [coberturaAdapter('build/logs/cobertura.xml')]
            }
        }
    }
    post {
        always {
            recordIssues([
            sourceCodeEncoding: 'UTF-8',
            enabledForFailure: true,
            aggregatingResults: true,
            blameDisabled: true,
            referenceJobName: 'repo-name/master',
            tools: [
                phpCodeSniffer(id: 'phpcs', name: 'CodeSniffer', pattern: 'build/logs/phpcs.checkstyle.xml', reportEncoding: 'UTF-8'),
                phpStan(id: 'phpstan', name: 'PHPStan', pattern: 'build/logs/phpstan.checkstyle.xml', reportEncoding: 'UTF-8'),
                phpCodeSniffer(id: 'phpcompat', name: 'PHP Compatibility', pattern: 'build/logs/phpcs-compat.checkstyle.xml', reportEncoding: 'UTF-8')
            ]
        ])
        }
    }
}
