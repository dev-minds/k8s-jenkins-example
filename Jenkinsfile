pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "master" || env.BRANCH_NAME == "develop" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "master" ? "prod" : "staging"}"
        VERSION_FETCH = readMavenPom().getVersion()
        VERSION = "${VERSION_FETCH}-${BUILD_NUMBER}"
        DOMAIN = 'localhost'
        REGISTRY = 'phelun/kotlas-ci'
        REGISTRY_CREDENTIAL = 'dockerhub-davidcampos'
    }
    agent any
    // agent {
    //     kubernetes {
    //         defaultContainer 'jnlp'
    //         yamlFile 'build.yaml'
    //     }
    // }
    stages {
        // stage('Build') {
        //     steps {
        //         container('maven') {
        //             sh 'mvn package'
        //         }
        //     }
        // }
        stage('Build') {
            steps {
                sh 'bash mvnw package'
            }
        }
        stage('Docker Build') {
            when {
                environment name: 'DEPLOY', value: 'false'
            }
            steps {
                // container('docker') {

                    sh "docker build -t ${REGISTRY}:${VERSION} ."
                // }
            }
        }
        stage('Docker Publish') {
            // when {
            //     environment name: 'DEPLOY', value: 'false'
            // }
            steps {
                // container('docker') {
                    // withDockerRegistry([credentialsId: "${REGISTRY_CREDENTIAL}", url: ""]) {
                    // }
                // }
                    withCredentials([usernamePassword(credentialsId: 'REGISTRY_CREDENTIAL', passwordVariable: 'reg_passwd', usernameVariable: 'reg_username')]) {
                        sh "docker login -u $reg_username -p $reg_passwd"
                        sh "docker push ${REGISTRY}:${VERSION}"
                    }
            }
        }
        stage('Kubernetes Deploy') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('helm') {
                    sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} --set domain=${DOMAIN} ${NAME} ./helm"
                }
            }
        }
    }
    post { 
        always { 
            echo 'Im done here, lets clean up workspace'
            cleanWs()
        }
    }
}