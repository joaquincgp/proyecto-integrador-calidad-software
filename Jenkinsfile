pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    options {
        disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout(true)
    }

    environment {
        MAVEN_OPTS = '-Djava.awt.headless=true'
        SERVIDOR_SONARQUBE = 'SonarQubeCloud'
        CLAVE_PROYECTO_SONAR = 'Mattair39_ProyectoIntegrador-CalidadDeSoftware'
        NOMBRE_PROYECTO_SONAR = 'ProyectoIntegrador-CalidadDeSoftware'
        ORGANIZACION_SONAR = 'mattair39'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Repositorio clonado correctamente desde la rama ${env.BRANCH_NAME}."
            }
        }

        stage('Validación de Entorno') {
            steps {
                bat 'git --version'
                bat 'java -version'
                bat 'mvn -version'
                echo 'Entorno validado correctamente.'
            }
        }

        stage('Build') {
            steps {
                echo "Compilando proyecto en la rama ${env.BRANCH_NAME}..."
                bat 'mvn -B clean compile'
                echo 'Build completado correctamente.'
            }
        }

        stage('Tests Unitarios') {
            steps {
                echo 'Ejecutando pruebas unitarias...'
                bat 'mvn -B test'
                echo 'Pruebas unitarias completadas.'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'target/surefire-reports/*'
                }
            }
        }

        stage('Análisis Estático en Paralelo') {
            parallel {
                stage('Checkstyle') {
                    steps {
                        echo 'Ejecutando análisis de estilo con Checkstyle...'
                        bat 'mvn -B checkstyle:check'
                        echo 'Checkstyle completado.'
                    }
                    post {
                        always {
                            archiveArtifacts allowEmptyArchive: true, artifacts: 'target/checkstyle-result.xml,target/checkstyle-checker.xml,target/site/checkstyle.html'
                        }
                    }
                }

                stage('PMD') {
                    steps {
                        echo 'Ejecutando análisis estático con PMD...'
                        bat 'mvn -B pmd:check'
                        echo 'PMD completado.'
                    }
                    post {
                        always {
                            archiveArtifacts allowEmptyArchive: true, artifacts: 'target/pmd.xml,target/site/pmd.html'
                        }
                    }
                }

                stage('JaCoCo') {
                    steps {
                        echo 'Generando reporte de cobertura con JaCoCo...'
                        bat 'mvn -B test jacoco:report'
                        echo 'JaCoCo completado.'
                    }
                    post {
                        always {
                            archiveArtifacts allowEmptyArchive: true, artifacts: 'target/site/jacoco/**'
                        }
                    }
                }
            }
        }

        stage('Análisis SonarQube') {
            steps {
                echo 'Ejecutando análisis SonarQube...'
                withSonarQubeEnv("${SERVIDOR_SONARQUBE}") {
                    bat "mvn -B sonar:sonar -Dsonar.projectKey=${CLAVE_PROYECTO_SONAR} -Dsonar.projectName=${NOMBRE_PROYECTO_SONAR} -Dsonar.organization=${ORGANIZACION_SONAR}"
                }
                echo 'Análisis SonarQube enviado correctamente.'
            }
        }

        stage('Cierre CI Develop') {
            when {
                branch 'develop'
            }
            steps {
                echo 'La rama develop ejecuta integración continua completa hasta análisis y validación, sin package ni deploy.'
            }
        }

        stage('Package') {
            when {
                branch 'main'
            }
            steps {
                echo 'Empaquetando artefacto JAR para main...'
                bat 'mvn -B package -DskipTests'
                echo 'Empaquetado completado.'
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'target/*.jar'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo 'Publicando artefacto para main...'
                bat '''
                    if not exist deploy mkdir deploy
                    copy /Y target\\*.jar deploy\\
                '''
                archiveArtifacts allowEmptyArchive: true, artifacts: 'deploy/*.jar'
                echo 'Deploy completado correctamente.'
            }
        }
    }

    post {
        success {
            echo "El pipeline finalizó correctamente para la rama ${env.BRANCH_NAME}."
        }
        failure {
            echo "El pipeline falló para la rama ${env.BRANCH_NAME}. Revisar logs de consola."
        }
        always {
            cleanWs()
        }
    }
}
