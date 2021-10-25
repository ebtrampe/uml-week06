pipeline {
    agent {
        kubernetes {
            yaml '''
spec:
  containers:
  - name: gradle
    image: gradle:6.3-jdk14
    command:
    - sleep
    args:
    - 30d
            '''
        }
    }
    stages {
        stage('Build Gradle') {
            steps {
                sh '''
                chmod +x gradlew
                ./gradlew test
                '''
            }
        }
        stage('debug') {
            steps {
                echo 'debugging'
                echo env.GIT_BRANCH
                echo env.GIT_LOCAL_BRANCH
            }
        }
        stage('Code Coverage') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main'
                }
            }
            steps {
                echo 'This test runs on the main branch only'
                sh '''
                ./gradlew jacocoTestCoverageVerification
                ./gradlew jacocoTestReport
                '''
            }
            post {
                success {
                    publishHTML (target: [ 
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        reportDir: './build/reports/jacoco/test/html', 
                        reportFiles: 'index.html', 
                        reportName: "JaCoCo Report" 
                    ])
                }
            }
        }
        stage('Checkstyle') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'origin/feature'
                }
            }
            steps {
                echo 'This test runs on both main and feature branches.'
                sh './gradlew checkstyleMain'
            }
            post {
                success {
                    publishHTML (target: [ 
                        reportDir: './build/reports/checkstyle', 
                        reportFiles: 'main.html', 
                        reportName: "JaCoCo Checkstyle report" 
                    ])
                }
            }
        }

    }
}