def isPR = env.CHANGE_ID != null

// Skip if this is NOT a PR
if (!isPR) {
    println "Skipping build for branch ${env.BRANCH_NAME} — not a PR."
    return
}


pipeline {
    agent any


    environment {

        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = 'learnjenkins'
        AWS_ECS_SERVICE = 'jenkins-app-service'
        AWS_ECS_TASK = 'LearnJenkinsApps-task-def'
        AWS_DOCKER_REGISTRY = '257394477950.dkr.ecr.us-east-1.amazonaws.com'
        APP_NAME = 'jenkinsapp'
    }

    stages {

        stage('Build') {
            // when {
            //     expression { return env.CHANGE_ID != null || env.BRANCH_NAME != null }
            // }
            agent {
                docker {
                    image 'node:20-slim'
                    reuseNode true
                }
            }
            
            environment {
                // Set a custom cache directory inside the container to avoid permission issues
                NPM_CONFIG_CACHE = '/tmp/.npm'
            }

            steps {
                sh'''
                npm ci
                npm run build
                '''
            }
        }

        stage('Test') {
            // when {
            //     expression { return env.CHANGE_ID != null || env.BRANCH_NAME != null }
            // }            
            agent {
                docker {
                    image 'node:20-slim'
                    reuseNode true
                }
            }            
            
            environment {
                // Set a custom cache directory inside the container to avoid permission issues
                NPM_CONFIG_CACHE = '/tmp/.npm'
            }

            steps {
                sh'''
                test -f build/index.html
                npm test

                '''
            }
        }

        stage('E2E') {
            // when {
            //     expression { return env.CHANGE_ID != null || env.BRANCH_NAME != null }
            // }            
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }            
            
            environment {
                // Set a custom cache directory inside the container to avoid permission issues
                NPM_CONFIG_CACHE = '/tmp/.npm'
            }

            steps {
                sh'''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html

                '''
            }
        }        


        // stage('Build Docker image') {
        //     agent {
        //         docker {
        //             image 'amazon/aws-cli'
        //             reuseNode true
        //             args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
        //         }
        //     }

        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        //         sh '''
        //         amazon-linux-extras install docker
        //         docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
        //         aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
        //         docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
        //         '''    
        //         }

        //     }
        // }

        // stage('Deploy to AWS') {
        //     agent {
        //         docker {
        //             image 'amazon/aws-cli'
        //             args "-u root --entrypoint=''"
        //         }
        //     }

        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        //         sh'''
        //         aws --version
        //         yum install jq -y
        //         sed -i "s/IMG_VERSION/$REACT_APP_VERSION/g" aws/task-definition-prod.json
        //         LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
        //         echo $LATEST_TD_REVISION
        //         aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK:$LATEST_TD_REVISION
        //         aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE
        //         '''
        //         }

        //     }
        // }        
                         
    }

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }


}
