pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github')
        IMAGE_NAME = 'ghcr.io/finkash-pty/frappe_docker_skm'
        RELEASE_VERSION = readFile('version').trim()
        FRAPPE_BRANCH = readFile('version-frappe').trim()
        ERPNEXT_BRANCH = readFile('version').trim()
        IMAGE_FRAPPE = "${IMAGE_NAME}:${RELEASE_VERSION}"
        IMAGE_BENCH = "${IMAGE_NAME}-bench:${RELEASE_VERSION}"
        PLATFORM = 'linux/amd64,linux/arm64'
        DOCKERFILE_FRAPPE = './images/production/Containerfile'
        DOCKERFILE_BENCH = './images/bench/Dockerfile'
        DOCKER_BUILD = "docker buildx build --platform $PLATFORM"
        DOCKER_CACHE_FROM = '--cache-from type=local,src=/tmp/.buildx-cache'
        DOCKER_CACHE_TO = '--cache-to type=local,dest=/tmp/.buildx-cache'
        DOCKER_PUSH_FRAPPE = "--push --tag $IMAGE_FRAPPE -f $DOCKERFILE_FRAPPE"
        DOCKER_PUSH_BENCH = "--push --tag $IMAGE_BENCH -f $DOCKERFILE_BENCH"
    }
    stages {
        stage('Login to image Repository') {
            steps {
                sh 'echo $GITHUB_TOKEN_PSW | docker login ghcr.io -u GITHUB_TOKEN_USR --password-stdin'
            }
        }
        stage('Prepare Builder') {
            steps {
                script {
                    sh 'export DOCKER_CLI_EXPERIMENTAL=enabled'
                    sh 'docker buildx use multiplatformbuilder'
                }
            }
        }
        stage('Decide on Cache Usage') {
            steps {
                script {
                    USE_CACHE = input(id: 'cache', message: 'Use Docker cache?', parameters: [booleanParam(defaultValue: true, description: 'Enable Docker cache during build', name: 'Use Cache')])
                }
            }
        }
        stage('Build Images') {
            steps {
                script {
                    def cacheOptions = USE_CACHE ? "$DOCKER_CACHE_FROM $DOCKER_CACHE_TO" : ""
                    sh "$DOCKER_BUILD $cacheOptions $DOCKER_PUSH_FRAPPE ."
                    sh "$DOCKER_BUILD $cacheOptions $DOCKER_PUSH_BENCH ."
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
