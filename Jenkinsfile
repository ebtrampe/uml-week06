pipeline {
    agent {
        kubernetes {
            yaml '''
            spec:
              containers:
              - name: gradle
                image: gradle:6.3-jdk14
            '''
        }
    }
    stages {
        stage('build') {
            steps {
                chmod +x gradlew
                ./gradlew test
            }
        }
        stage('debug') {
            steps {
                echo env.GIT_BRANCH
                echo env.GIT_LOCAL_BRANCH
            }
        }
        stage('feature') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/feature/*'
                }
            }
            steps {
                echo "I am a feature branch"
                echo 'Running Checkstyle'
                ./gradlew checkstyleMain
                publishHTML (target: [
                    reportDir: 'Chapter08/sample1/build/reports/jacoco/checkstyle',
                    reportFiles: 'main.html',
                    reportName: "JaCoCo Checkstyle report"
                ])
            }
        }
        stage('main') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/main"
                }
            }
            steps {
                echo "I am a main branch"
                echo 'Running CodeCoverage test'
                ./gradlew jacocoTestCoverageVerification
                ./gradlew jacocoTestReport
                publishHTML (target: [ 
                    reportDir: 'Chapter08/sample1/build/reports/jacoco/test/html',
                    reportFiles: 'index.html',
                    reportName: "JaCoCo Report"
                ])
                echo 'Running Checkstyle'
                ./gradlew checkstyleMain
                publishHTML (target: [ 
                    reportDir: 'Chapter08/sample1/build/reports/jacoco/checkstyle',
                    reportFiles: 'main.html',
                    reportName: "JaCoCo Checkstyle report"
                ])
            }
        }
    }
}
