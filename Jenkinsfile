pipeline {
    agent {
        label 'master'
    }
    stages {
        stage('Prepare') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend') {
                    echo 'Preparing...'
                    sh 'npm install'
                    echo 'Prepared'
                }
            }
        }
        stage('Build') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend'){
                    sh 'npm run build'
                }
            }
        }
        stage('Static Analysis') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend'){
                    sh 'npm run lint'
                }
            }
        }
        stage('Unit Test') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend') {
                    sh 'npm run test'
                }
            }
        }
        stage('e2e Test') {
            steps {
                dir('code/frontend') {
                    sh 'docker-compose -f docker-compose-e2e.yml build'
                    script {
                        sh 'docker-compose -f docker-compose-e2e.yml up e2e'
                        status_code = sh ( script: "docker inspect code_e2e_1 --format='{{.State.ExitCode}}'", returnStdout: true).trim();
                        if (status_code == '1') {
                            error('e2e test failed.')
                        }
                    }
                }
            }
            post {
                always {
                    echo 'Cleanup'
                    dir('code/frontend') {
                        sh 'docker-compose -f docker-compose-e2e.yml down --rmi=all -v'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                dir('code/frontend') {
                    sh 'docker-compose -f docker-compose.yml build'
                    sh 'docker-compose -f docker-compose.yml up -d frontend backend'   
                }
                echo 'Deploy'
            }
        }
    }
}
