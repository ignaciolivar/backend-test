pipeline {
    agent any

    stages {

        stage("Procesando aplicacion NodeJS") {
            agent { 
                docker {
                    image "node:22"
                }
            }

            stages {

                stage('inicio pipeline') {
                    steps {
                        sh 'echo "Iniciando pipeline..."'
                    }
                }

                stage('dependencias') {
                    steps {
                        sh 'echo "Instalando dependencias..."'
                        sh 'npm install'
                    }
                }

                stage('Lint de codigo') {
                    steps {
                        sh 'echo "Ejecutando linter..."'
                        sh 'npm run lint'
                    }
                }

                stage('Ejecutando test y coverage') {
                    steps {
                        sh 'echo "Ejecutando tests..."'
                        sh 'npm run test:cov'
                    }
                }

                stage('Ejecutando build') {
                    steps {
                        sh 'echo "Construyendo aplicación..."'
                        sh 'npm run build'
                    }
                }
            }
        }

        stage('Build docker image') {
            steps {
                sh 'echo "Construyendo imagen Docker..."'
                sh 'docker build -t backend-node .'

                script {

                    // ---- Push a Docker Hub ----
                    docker.withRegistry("https://index.docker.io/v1/", "credencial-dh") {
                        sh 'docker tag backend-node iolivaresv/backend-node:latest'
                        sh "docker tag backend-node iolivaresv/backend-node:${env.BUILD_NUMBER}"

                        sh 'docker push iolivaresv/backend-node:latest'
                        sh "docker push iolivaresv/backend-node:${env.BUILD_NUMBER}"
                    }

                    //// ---- Push a GitHub Packages ----
                    //docker.withRegistry("https://ghcr.io", "credencial-gh") {
                    //    sh 'docker tag backend-node ghcr.io/iolivaresv/backend-node:latest'
                    //    sh "docker tag backend-node ghcr.io/iolivaresv/backend-node:${env.BUILD_NUMBER}"
//
                    //    sh 'docker push ghcr.io/iolivaresv/backend-node:latest'
                    //    sh "docker push ghcr.io/iolivaresv/backend-node:${env.BUILD_NUMBER}"
                    //}
                }
            }
        }
        stage('Despliegue continuo') {
            agent{
                docker{
                    image 'alpine/k8s:1.32.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: '+´']){
                     sh "kubectl -n devops set image deployments backend-dp backend=iolivares/backend-node:${env.BUILD_NUMBER}"
                }
            }
        }
        stage('fin pipeline') {
            steps {
                echo 'Pipeline finalizado correctamente.'
            }
        }
    }
}
