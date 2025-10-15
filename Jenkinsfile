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

        stage('SAST-TEST')
        {
            agent any
            steps
            {
                script
                {
                    snykSecurity(
                        snykInstallations: 'Snyk-installations',
                        snykTokenId: 'Snyk-API-token',
                        severity: 'critical'
                    )
                }
            }
        }

        stage('BUILD-AND-TAG')
        {
            agent { label 'appserver'}
            steps
            {
                script
                {
                    // Build Docker image using Jenkins Docker Pipeline API
                    echo "Building Docker image ${IMAGE_NAME}..."
                    app = docker.build("${IMAGE_NAME}")
                    app.tag("latest")
                }
            }
        }

        stage('PUSH-TO-DOCKERHUB')
        {
            agent { label 'appserver'}
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
            agent { label 'appserver'}
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
