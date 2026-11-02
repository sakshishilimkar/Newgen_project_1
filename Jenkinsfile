pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-1'
        S3_BUCKET = 'uyyfghngjhkllllj'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    git --version
                    java -version
                    mvn -version
                    aws --version
                '''
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Verify JAR') {
            steps {
                sh '''
                    ls -lh target/*.jar
                    test -n "$(find target -maxdepth 1 -type f -name '*.jar' ! -name '*.original' -print -quit)"
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                sh '''
                    JAR_FILE=$(find target -maxdepth 1 -type f -name '*.jar' ! -name '*.original' -print -quit)

                    if [ -z "$JAR_FILE" ]; then
                        echo "No JAR file found!"
                        exit 1
                    fi

                    aws s3 cp "$JAR_FILE" \
                    "s3://${S3_BUCKET}/${BRANCH_NAME}/build-${BUILD_NUMBER}/$(basename "$JAR_FILE")"

                    aws s3 cp "$JAR_FILE" \
                    "s3://${S3_BUCKET}/${BRANCH_NAME}/latest.jar"
                '''
            }
        }
    }

    post {
        success {
            echo 'Build successful and artifact uploaded to S3!'
        }

        failure {
            echo 'Build failed. Check the console output.'
        }
    }
}