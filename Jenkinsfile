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
        stage('build') {
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
        stage('feature') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/feature'
                }
            }
            steps {
                echo 'I am a feature branch. Only checkstyles will be executed in this branch.'
                sh './gradlew checkstyleMain'
                publishHTML (target: [ 
                    reportDir: './build/reports/jacoco/checkstyle', 
                    reportFiles: 'main.html', 
                    reportName: "JaCoCo Checkstyle report" 
                ])
            }
        }
        stage('main') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main'
                }
            }
            steps {
                echo 'I am the main branch. I run all the tests.'
                sh '''
                ./gradlew jacocoTestCoverageVerification
                ./gradlew jacocoTestReport
                '''
                publishHTML (target: [ 
                    reportDir: 'Chapter08/sample1/build/reports/jacoco/test/html', 
                    reportFiles: 'index.html', 
                    reportName: "JaCoCo Report" 
                ])
                sh './gradlew checkstyleMain'
                publishHTML (target: [ 
                    reportDir: './build/reports/jacoco/checkstyle', 
                    reportFiles: 'main.html', 
                    reportName: "JaCoCo Checkstyle report" 
                ])
            }
        }
    }
}