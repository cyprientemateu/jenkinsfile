pipeline {
    agent {
        label ''
    }
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HUB_REGISTRY = "cyprientemateu"
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '2'))
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout (time: 15, unit: 'MINUTES')
        timestamps()
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string(name: 'SONAR_TAG', defaultValue: '5.0.1.3006', description: '')
        string (name: 'AUTH_IMAGE_TAG', defaultValue: 'v1.0.0', description: '')
        string (name: 'DB_IMAGE_TAG', defaultValue: 'v1.0.0', description: '')
        string (name: 'REDIS_IMAGE_TAG', defaultValue: 'v1.0.0', description: '')
        string (name: 'UI_IMAGE_TAG', defaultValue: 'v1.0.0', description: '')
        string (name: 'WEATHER_IMAGE_TAG', defaultValue: 'v1.0.0', description: '')
    }
    stages {
        stage ('Sanity Check') {
            steps {
                script {
                   sanityChecks()
                }
            }
        }
        stage ('Checkout') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${env.BRANCH_NAME}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch']],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            url: 'https://github.com/cyprientemateu/jenkins-test.git',
                            credentialsId: 'github-auth'
                        ]]
                    ])
                }
            }
        }
        stage('Open sonar-project.properties') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        sh 'cat sonar-project.properties'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        withSonarQubeEnv('SonarScanner') {
                            sh "sonar-scanner"
                        }
                    }
                }
            }
        }
        stage('Build auth') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code/auth") {
                    script {
                        sh """
                            sudo docker build -t ${env.DOCKER_HUB_REGISTRY}/sixfure-auth:${params.AUTH_IMAGE_TAG} .
                            sudo docker images
                        """
                    }
                }
            }
        }
        stage('Build db') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code/db") {
                    script {
                        sh """
                            sudo docker build -t ${env.DOCKER_HUB_REGISTRY}/sixfure-db:${params.DB_IMAGE_TAG} .
                            sudo docker images
                        """
                    }
                }
            }
        }
        stage('Build redis') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code/redis") {
                    script {
                        sh """
                            sudo docker build -t ${env.DOCKER_HUB_REGISTRY}/sixfure-redis:${params.REDIS_IMAGE_TAG} .
                            sudo docker images
                        """
                    }
                }
            }
        }
        stage('Build ui') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code/ui") {
                    script {
                        sh """
                            sudo docker build -t ${env.DOCKER_HUB_REGISTRY}/sixfure-ui:${params.UI_IMAGE_TAG} .
                            sudo docker images
                        """
                    }
                }
            }
        }
        stage('Build weather') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code/weather") {
                    script {
                        sh """
                            sudo docker build -t ${env.DOCKER_HUB_REGISTRY}/sixfure-weather:${params.WEATHER_IMAGE_TAG} .
                            sudo docker images
                        """
                    }
                }
            }
        }
        stage("Login Into TCC Docker Hub"){
             steps {
              withCredentials([
                usernamePassword(credentialsId: 'jenkins-dockerhub-token', 
                usernameVariable: 'DOCKER_HUB_USERNAME', 
                passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                  sh """
                    sudo docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD}
                  """
                  
                }
            }
        }
        stage('Pushing Into TCC Docker Hub') {
            steps {
                script {
                    dir("${WORKSPACE}/tcc-weather-app") {
                        sh """
                            sudo docker push ${env.DOCKER_HUB_REGISTRY}/sixfure-auth:${params.AUTH_IMAGE_TAG}
                            sudo docker push ${env.DOCKER_HUB_REGISTRY}/sixfure-db:${params.DB_IMAGE_TAG}
                            sudo docker push ${env.DOCKER_HUB_REGISTRY}/sixfure-redis:${params.REDIS_IMAGE_TAG}
                            sudo docker push ${env.DOCKER_HUB_REGISTRY}/sixfure-ui:${params.UI_IMAGE_TAG}
                            sudo docker push ${env.DOCKER_HUB_REGISTRY}/sixfure-weather:${params.WEATHER_IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage('Set variables') {
            // agent {
            //     label ''
            // }
            steps {
                script {
                     dir("${WORKSPACE}/tcc-weather-app/docker-stack") {
                        withCredentials([
                        usernamePassword(credentialsId: 'weather-app-redis-cred', 
                        usernameVariable: 'REDIS_USERNAME', 
                        passwordVariable: 'REDIS_PASSWORD'),

                        string(credentialsId: 'weather-app-api-key', 
                        variable: 'API_TOKEN'),

                        string(credentialsId: 'weather-app-mysql-root-password', 
                        variable: 'MYSQL_ROOT_PASSWORD'),

                        string(credentialsId: 'weather-app-mysql-password', 
                        variable: 'MYSQL_PASSWORD')]) {
                            settingUpVariable()
                        }
                     }
                }
            }
        }
        stage("Pulling Images From Docker Hub"){
            // agent {
            //     label ''
            // }
            steps {
              withCredentials([
                usernamePassword(credentialsId: 'jenkins-dockerhub-token', 
                usernameVariable: 'DOCKER_HUB_USERNAME', 
                passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                  sh """
                    sudo docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD}
                  """
                  pullImages()
                }
            }
        }
        stage('Deploying The Application') {
            // agent {
            //     label ''
            // }
            steps {
                script {
                     dir("${WORKSPACE}/tcc-weather-app/docker-stack") {
                        sh """
                            sudo docker swarm init
                            sleep 10
                            sudo docker stack deploy -c docker-compose.yml weather-app
                        """
                     }
                }
            }
        }
        stage('Check Services') {
            // agent {
            //     label ''
            // }
            steps {
                script {
                    sh """
                        sleep 30
                        sudo docker stack ls
                        sudo docker service ls
                        sudo docker ps
                    """
                }
            }
        }
        stage('Clean Up') {
            steps {
                script {
                    dir("${WORKSPACE}/tcc-weather-app") {
                        sh """
                            sudo rm -rf *
                        """
                    }
                }
            }
        }
    }
    post {
        
        success {
            slackSend color: '#2EB67D',
            channel: 'tcc-lab',  
            message: "*tcc-weather-app Project Build Status*" +
            "\n Project Name: tcc-weather-app" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *SUCCESS*" +
            "\n Build url : ${env.BUILD_URL}"
        }
        failure {
            slackSend color: '#E01E5A', 
            channel: 'tcc-lab',  
            message: "*tcc-weather-app Project Build Status*" +
            "\n Project Name: tcc-weather-app" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *FAILED*" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }
        unstable {
            slackSend color: '#ECB22E',
            channel: 'tcc-lab', 
            message: "*tcc-weather-app Project Build Status*" +
            "\n Project Name: tcc-weather-app" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *UNSTABLE*" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }   
    }
}

def settingUpVariable() {
    sh """
    sed -i "s|DB_IMAGE_TAG|${params.DB_IMAGE_TAG}|g" docker-compose.yml
    sed -i "s|REDIS_IMAGE_TAG|${params.REDIS_IMAGE_TAG}|g" docker-compose.yml
    sed -i "s|UI_IMAGE_TAG|${params.UI_IMAGE_TAG}|g" docker-compose.yml
    sed -i "s|WEATHER_IMAGE_TAG|${params.WEATHER_IMAGE_TAG}|g" docker-compose.yml
    sed -i "s|AUTH_IMAGE_TAG|${params.AUTH_IMAGE_TAG}|g" docker-compose.yml
    
    sed -i "s|WEATHER_APP_REDIS_PASSWORD_USERNAME|${REDIS_USERNAME}|g" docker-compose.yml
    sed -i "s|WEATHER_APP_REDIS_PASSWORD|${REDIS_PASSWORD}|g" docker-compose.yml
    sed -i "s|WEATHER_API-TOKEN|${API_TOKEN}|g" docker-compose.yml
    sed -i "s|WEATHER_APP_MYSQL_ROOT_PASSWORD|${MYSQL_ROOT_PASSWORD}|g" docker-compose.yml
    sed -i "s|WEATHER_APP_MYSQL_PASSWORD|${MYSQL_PASSWORD}|g" docker-compose.yml
    cat docker-compose.yml
    """
}

def pullImages() {
    sh """
    sudo docker pull ${env.DOCKER_HUB_REGISTRY}/sixfure-db:${params.DB_IMAGE_TAG}
    sudo docker pull ${env.DOCKER_HUB_REGISTRY}/sixfure-redis:${params.REDIS_IMAGE_TAG}
    sudo docker pull ${env.DOCKER_HUB_REGISTRY}/sixfure-ui:${params.UI_IMAGE_TAG}
    sudo docker pull ${env.DOCKER_HUB_REGISTRY}/sixfure-weather:${params.WEATHER_IMAGE_TAG}
    sudo docker pull ${env.DOCKER_HUB_REGISTRY}/sixfure-auth:${params.AUTH_IMAGE_TAG}
    """
}

def sanityChecks() {
    if (params.BRANCH_NAME.isEmpty()) {
        println('Parameter BRANCH_NAME is not set')
        sh """
            exit 2
        """
    }
}        
