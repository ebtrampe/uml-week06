pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsUser: 0
  containers:
  - name: gradle
    image: gradle:6.3-jdk14
    command:
    - sleep
    args:
    - 30d
    volumeMounts:
    - name: shared-storage
      mountPath: '/mnt'
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
    - name: shared-storage
      mountPath: '/mnt'
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: shared-storage
    persistentVolumeClaim:
      claimName: jenkins-pv-claim
  - name: kaniko-secret
    secret:
      secretName: dockercred
      items:
      - key: .dockerconfigjson
        path: config.json
            '''
        }
    }
    environment {
        NAME = "${env.GIT_BRANCH == 'origin/main' ? 'calculator' : 'calculator-feature'}"
        TAG = "${env.GIT_BRANCH == 'origin/main' ? '1.0' : '0.1'}"
    }
    stages {
        stage('Make gradle binary executable') {
            when {
                expression {
                    return env.GIT_BRANCH != 'origin/playground'
                }
            }
            steps {
                echo env.NAME
                echo env.TAG
                sh '''
                chmod +x gradlew
                ./gradlew test
                '''
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
        stage('Build Java Image') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'origin/feature' 
                }
            }
            steps {
                sh '''
                pwd
                ./gradlew build
                ls -al ./build/libs/
                cp ./build/libs/calculator-0.0.1-SNAPSHOT.jar ${pwd}
                ls -al ${pwd}
                '''
            }
            post {
                success {
                    container('kaniko') {
                        sh '''
                        pwd
                        ls -al /mnt
                        ls -al ${pwd}
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                        /kaniko/executor --context `pwd` --destination ebtrampe/${NAME}:${TAG}
                        '''
                    }
                }
            }
        }
    }
}