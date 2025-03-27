pipeline {
    agent any
    tools {
        maven 'M2_HOME'
    }
    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the Git repository'
                git branch: 'master', url: 'https://github.com/yadagiri7013/insurance-project.git'
            }
        }
        stage('Create a Package') {
            steps {
                echo 'Creating a package using Maven'
                sh 'mvn clean package'
            }
        }
        stage('Publish test reports') {
            steps {
                echo 'Publishing test reports (HTML) from surefire reports'
                publishHTML([allowMissing: false, 
                             alwaysLinkToLastBuild: false, 
                             keepAll: false, 
                             reportDir: '/var/lib/jenkins/workspace/Capstone-Project-Live-Demo/target/surefire-reports', 
                             reportFiles: 'index.html', 
                             reportName: 'HTML Report', 
                             reportTitles: '', 
                             useWrapperFileDirectly: true])
            }
        }
        stage('Create a Docker image from the Package Insure-Me.jar file') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t yadagiri14/insurance:latest .'
            }
        }
        stage('Login to Dockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-id', 
                                                  passwordVariable: 'dockerPass', 
                                                  usernameVariable: 'dockerUser')]) {
                    echo 'Logging in to Dockerhub'
                    sh 'docker login -u ${dockerUser} -p ${dockerPass}'
                }
            }
        }
        stage('Push the Docker image') {
            steps {
                script {
                    // Define the tag dynamically based on the build number
                    def tagname = "v${env.BUILD_NUMBER}"  // Using the Jenkins environment variable BUILD_NUMBER
                    echo "Pushing Docker image with tag: ${tagname}"
                    sh "docker push yadagiri14/insurance:${tagname}"
                }
            }
        }
        stage('Ansible config and Deployment') {
            steps {
                echo 'Running Ansible Playbook for deployment'
                ansiblePlaybook credentialsId: 'ansible-ssh', 
                                disableHostKeyChecking: true, 
                                installation: 'ansible', 
                                inventory: '/etc/ansible/hosts', 
                                playbook: 'ansible-playbook.yml', 
                                vaultTmpPath: ''
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
