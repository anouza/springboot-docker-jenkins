pipeline {
    agent any

    environment {
        MAVEN_ARGS=" -e clean install"
        dockerContainerName="bookapi_${params.ENV}"
        dockerImageName="bookapi_api_${params.ENV}"
        SPRING_PROFILES_ACTIVE=${params.ENV}
    }

    parameters {
        choice(name: 'ENV', choices:['staging','production'], description: "Select Environment (staging, production)")
    }

    stages {
        stage('Initialize'){
            steps {
                script {
                    // default value
                    if(${!params.ENV}) params.ENV = 'staging'
                }
            }
        }
        stage('Build'){
            steps {
                withMaven(maven: 'maven_home'){
                    sh "mvn ${MAVEN_ARGS}"
                }
            }
        }
        stage('Clean container'){
            steps {
                sh 'docker ps -f name=${dockerContainerName} -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -f name=${dockerContainerName} -q | xargs -r docker container rm'
                sh 'docker images -q --filter=reference={dockerImageName} | xargs --no-run-if-empty docker rmi -f'
            }
        }
        stage('Start docker-compose'){
            steps {
                // sh 'docker-compose up -d --build'
                sh """
                SPRING_PROFILES_ACTIVE=${SPRING_PROFILES_ACTIVE} docker-compose -f docker-compose-${params.ENV}.yml up -d --build
                """
            }
        }
    }
}