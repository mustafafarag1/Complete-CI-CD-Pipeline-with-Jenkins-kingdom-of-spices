#!/usr/bin/env groovy



pipeline {
    agent any
    tools {
        maven 'Maven'
    }



    
    stages {
        
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    
                }
            }
        }
        stage('build app') {
            steps {
               script {
                   echo "building the application..."
                   sh 'mvn clean package'
               }
            }
        }
        
        
        stage('Testing') {
            steps {
               script {
                   echo "Testing the application..."
                   
               }
            }
        }
        
        
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentails', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'docker build -t ahmedyoussef527/demo-app:${IMAGE_NAME} .'
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh 'docker push ahmedyoussef527/demo-app:${IMAGE_NAME}'
                    }
                }
            }
        } 


        stage('deploy') {
            
            when {
                expression {BRANCH_NAME == 'main' }
            }
           
            steps {
 
                script {
                    echo 'deploying docker image to ec2 ....'
                    def shellCmd = "bash ./server-cmds.sh ahmedyoussef527/demo-app:${IMAGE_NAME}"
                    def ec2Instance = "ubuntu@54.202.195.125"
                    sshagent ( ['ec2-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ubuntu"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ubuntu"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }
        
  
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'git config user.email "jenkins@example.com"'
                        sh 'git config user.name "Jenkins"'
                        
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        
                        
                        
                     
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/mustafafarag1/Complete-CI-CD-Pipeline-with-Jenkins-kingdom-of-spices.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh "git push origin HEAD:${BRANCH_NAME}"
                    }
                }
            }
        }
    }
}
