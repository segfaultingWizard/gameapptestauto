pipeline
{
    agent any
    environment
    {
        //Docker Hub
        DOCKERHUB_CREDENTIALS ='cybr-3120'
        IMAGE_NAME ='segfaultingwizard/gameapptestauto:latest'
    }

    stages
    {
        stage('Cloning Git')
        {
            steps
            {
                checkout scm
            }
        }

        stage('SAST')
        {
            steps
            {
                sh 'echo running SAST scan with SNYK...'
            }
        }

        stage('BUILD-AND-TAG')
        {
            agent { label 'app-server'}
            steps
            {
                script
                {
                    // Build Docker image using Jenkins Docker Pipeline API
                    echo "Building Docker image ${IMAGE_NAME}..."
                    app = docker.Build{"${IMAGE_NAME}"}
                    app.tag("latest")
                }
            }
        }

        stage('PUSH-TO-DOCKERHUB')
        {
            agent { label 'app-server'}
            steps
            {
                script
                {
                    // Push Docker image using Jenkins Docker Pipeline API
                    echo "Push Docker image ${IMAGE_NAME}:latest to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKERHUB_CREDENTIALS}") {
                        app.push("latest")
                    }
                }
            }
        }

        stage('DAST')
        {
            steps
            {
                sh 'echo running DAST scan...'
            }
        }

        stage('DEPLOYMENT')
        {
            agent { label 'app-server'}
            steps
            {
                echo 'Starting deployment using docker-compose...'
                script
                {
                    dir("${WORKSPACE}")
                    {
                        sh '''
                            docker-compose down
                            docker-compose up -d
                            docker ps
                        '''
                    }
                }
                echo 'Deployment completed successfully!'
            }

        }

    }
}
