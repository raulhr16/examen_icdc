pipeline {
    environment {
        IMAGEN = "raulhr16/exicdc"
        USUARIO = 'docker'
    }
    agent none
    stages {
        stage("Comprobacion del proyecto") {
            agent {
                docker { image 'python:3'
                args '-u root:root'
                }
            }
            stages {
                stage('Clonando') {
                steps {
                    git branch:'main',url:'https://github.com/raulhr16/examen_icdc.git'
                    }
                }
                stage('Instalando') {
                steps {
                    sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test') {
                steps {
                    sh 'pytest app/test_app.py'
                    }
                }
            }
        }
        stage("Creación de la imagen docker") {
            agent any
            stages {
                stage('Clonando') {
                steps {
                    git branch:'main',url:'https://github.com/raulhr16/examen_icdc.git'
                    }
                }
                stage('Build') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        script {
                            docker.withRegistry( '', USUARIO ) {
                                newApp.push()
                            }
                        }
                    }
                }
                stage('Clean Up') {
                    steps {
                        sh "docker rmi $IMAGEN:$BUILD_NUMBER"
                        }
                }
            }
        }
        stage ('Despliegue') {
            agent any
            stages {
                stage ('Deploy en el VPS'){
                    steps{
                        sshagent(credentials : ['vps']) {
                            sh """
                                ssh -o StrictHostKeyChecking=no debian@vps.raulhr.site "
                                cd examen_icdc &&
                                git pull &&
                                sed -i 's|image: .*|image: ${IMAGEN}:${BUILD_NUMBER}|' docker-compose.yaml &&
                                docker-compose down &&
                                docker pull "$IMAGEN:$BUILD_NUMBER" &&
                                docker-compose up -d &&
                                docker image prune -f"
                            """
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            emailext(
                to: 'alertas.snort.raulhr@gmail.com',
                subject: "Pipeline IC: ${currentBuild.fullDisplayName}",
                body: "Se ha generado la imagen $IMAGEN:$BUILD_NUMBER. El estado del despliegue es ${currentBuild.result} "
            )
        }
    }
}
