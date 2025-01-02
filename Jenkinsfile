pipeline {
    agent any
    environment {
        TEST_URL = 'http://localhost/laundry3/'
        STAGING_URL = 'http://localhost/laundry3/'
        STAGING_PATH = 'C:\\xampp\\htdocs\\laundry3'
        PHPCS_PATH = 'C:\\Users\\Geral\\AppData\\Roaming\\Composer\\vendor\\bin\\phpcs.bat'
        PHPCBF_PATH = 'C:\\Users\\Geral\\AppData\\Roaming\\Composer\\vendor\\bin\\phpcbf.bat'
    }
    stages {
        stage('Syntax Check') {
            steps {
                echo 'Checking PHP syntax...'
                bat 'php -l index.php'
            }
        }
        stage('Fix Formatting Issues') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    echo 'Fixing formatting issues with PHPCBF...'
                    bat """
                    ${PHPCBF_PATH} --standard=PSR12 .
                    
                    """
                }
            }
        }
        stage('Check Formatting (PHPCS)') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    echo 'Running static code analysis with PHPCS...'
                    bat """
                    ${PHPCS_PATH} --standard=PSR12 .
                    || exit /b 0
                    """
                }
            }
        }
        stage('Report Formatting Issues') {
            steps {
                echo 'Reporting remaining formatting issues...'
                bat """
                ${PHPCS_PATH} --standard=PSR12 .
                exit /b 0
                """
            }
        }
        stage('Commit Changes') {
            steps {
                bat """
                git config user.name "geralnb"
                git config user.email "shirometeora@gmail.com"
                git add .
                git commit -m "Auto-fix formatting issues using PHPCBF"
                git push origin dev
                """
            }
        }
        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                bat 'phpunit --configuration phpunit.xml'
            }
        }
        stage('DAST - Dynamic Application Security Testing') {
            steps {
                echo 'Running DAST scan...'
                bat """
                zap-cli quick-scan --self-contained --start-options '-config api.disablekey=true' ${TEST_URL}
                """
            }
        }
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying PHP application to XAMPP staging environment...'
                bat """
                if not exist ${STAGING_PATH} mkdir ${STAGING_PATH}
                xcopy /E /I /Y . ${STAGING_PATH}
                """
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}