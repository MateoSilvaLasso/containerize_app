pipeline {
    agent none

    tools {
        
        gradle 'Gradle3'
        maven "M3"
        jdk "OpenJDK-17"
    }

    stages {
        stage('Preparation') {
            agent { label 'node1' } 
            steps {
                checkout scm
            }
        }

        stage('Build') {
            
            agent { label 'node1' } 
           
                steps {
                    sh 'mvn spring-javaformat:apply'
                    // Ejecutar Gradle para compilar y ejecutar pruebas unitarias
                    sh "mvn clean install -DskipTests=true"

                    sh 'mvn test'
                }

                post {
                    success {
                        // Publicar resultados de las pruebas unitarias
                        junit '**/target/surefire-reports/*.xml'
                        // Almacenar los artefactos generados
                        archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
                        // Stash del archivo JAR para uso en etapas posteriores
                        stash includes: 'target/*.jar', name: 'app-jar'
                    }
                }
            }
                
            
            
        }

        stage('Javadoc Generation') {
            agent { label 'node1' }
            steps {
                
                sh 'mvn spring-javaformat:apply'

                sh 'mvn javadoc:javadoc'
            }
            post {
               success {
                   archiveArtifacts artifacts: '**/target/site/apidocs/**', allowEmptyArchive: true // Almacenar la documentación generada
               }
            }
        }

        stage('Deploy and Integration Tests') {
            agent { label 'node2' } 
            steps {
                // Recuperar el archivo JAR del nodo1
                unstash 'app-jar'

                script {
                    // Construir la imagen Docker usando el Dockerfile del repositorio
                    sh 'docker build -t my-app .'
                    
                    sh 'docker run -d -p 8081:8081 --name my-app-container my-app'
                }

                // Esperar a que la aplicación esté lista antes de ejecutar las pruebas de integración
                sleep time: 15, unit: 'SECONDS'

                // Ejecutar pruebas de integración
                sh 'mvn verify -Pintegration-tests'
            }
            post {
                always {
                    script {
                        // Detener y eliminar el contenedor Docker después de las pruebas
                        sh 'docker stop my-app-container || true'
                        sh 'docker rm my-app-container || true'
                    }
                }
                success {
                    // Publicar resultados de las pruebas de integración
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }

        
    }
}