pipeline {
    agent any

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Docker image tag to build (e.g. v1.0, latest, build-23)'
        )
        choice(
            name: 'GIT_BRANCH',
            choices: ['main', 'develop', 'release'],
            description: 'Git branch to checkout and build from'
        )
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run Maven test stage'
        )
        booleanParam(
            name: 'RUN_CONTAINER',
            defaultValue: true,
            description: 'Run the Docker container after build'
        )
        choice(
            name: 'DEPLOY_ENV',
            choices: ['DEV', 'QA', 'PROD'],
            description: 'Target deployment environment'
        )
    }

    environment {
        // Environment variables: computed/derived values, not user-chosen at trigger time
        IMAGE_NAME   = 'myorg/sample-app'
        FULL_IMAGE   = "${IMAGE_NAME}:${params.IMAGE_TAG}"
        CONTAINER_NAME = 'sample-app-container'
    }

    stages {

        stage('Show Build Parameters') {
            steps {
                echo "===================================="
                echo " Build Parameter Summary"
                echo "===================================="
                echo " IMAGE_TAG      : ${params.IMAGE_TAG}"
                echo " GIT_BRANCH     : ${params.GIT_BRANCH}"
                echo " RUN_TESTS      : ${params.RUN_TESTS}"
                echo " RUN_CONTAINER  : ${params.RUN_CONTAINER}"
                echo " DEPLOY_ENV     : ${params.DEPLOY_ENV}"
                echo " FULL_IMAGE     : ${env.FULL_IMAGE}"
                echo "===================================="
                // Never echo secrets/passwords/credentials here even if added later
            }
        }

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.GIT_BRANCH}"
                git branch: "${params.GIT_BRANCH}",
                    url: 'https://github.com/vkathiravan001-lang/getting-started-app.git'
            }
        }

        stage('Maven Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                echo "Running Maven tests..."
                sh 'mvn test'
            }
        }

        stage('Skip Tests Notice') {
            when {
                expression { return !params.RUN_TESTS }
            }
            steps {
                echo "RUN_TESTS = false -> Maven test stage skipped."
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image: ${env.FULL_IMAGE}"
                sh "docker build -t ${env.FULL_IMAGE} ."
            }
        }

        stage('Docker Run Container') {
            when {
                expression { return params.RUN_CONTAINER }
            }
            steps {
                echo "Stopping/removing any old container..."
                sh """
                    docker rm -f ${env.CONTAINER_NAME} || true
                    docker run -d --name ${env.CONTAINER_NAME} -p 8080:80 ${env.FULL_IMAGE}
                """
            }
        }

        stage('Skip Container Notice') {
            when {
                expression { return !params.RUN_CONTAINER }
            }
            steps {
                echo "RUN_CONTAINER = false -> container run stage skipped."
            }
        }

        stage('Deploy to DEV') {
            when {
                expression { return params.DEPLOY_ENV == 'DEV' }
            }
            steps {
                echo "Deploying ${env.FULL_IMAGE} to DEV environment..."
                // dev-specific deploy commands here
            }
        }

        stage('Deploy to QA') {
            when {
                expression { return params.DEPLOY_ENV == 'QA' }
            }
            steps {
                echo "Deploying ${env.FULL_IMAGE} to QA environment..."
                // qa-specific deploy commands here
            }
        }

        stage('Deploy to PROD') {
            when {
                expression { return params.DEPLOY_ENV == 'PROD' }
            }
            steps {
                echo "Deploying ${env.FULL_IMAGE} to PROD environment..."
                // add manual approval / stricter checks here for real PROD use
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully for branch ${params.GIT_BRANCH}, env ${params.DEPLOY_ENV}."
        }
        failure {
            echo "Pipeline failed. Check the stage logs above."
        }
    }
}
