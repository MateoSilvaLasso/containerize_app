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
            parallel{
                steps {
                    sh 'mvn spring-javaformat:apply'
                    // Ejecutar Gradle para compilar y ejecutar pruebas unitarias
                    sh "mvn clean install -DskipTests=true"

                    sh 'mvn test'
                }

                steps {
                    sh 'mvn javadoc:javadoc'
                }
            }
            
        }

        
    }
}