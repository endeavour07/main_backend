pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
        jdk 'JAVA_HOME'
    }

    environment {
        TOMCAT_HOME = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1_Tomcat10.26"
        BACKUP_ROOT = "D:\\Backups"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changed Services') {
            steps {
                script {
                    def diff = bat(
                        script: 'git diff --name-only origin/main...HEAD',
                        returnStdout: true
                    ).trim()

                    env.SERVICE1_CHANGED = diff.contains("service-1/") ? "true" : "false"
                    env.SERVICE2_CHANGED = diff.contains("service-2/") ? "true" : "false"

                    echo "service-1 changed: ${env.SERVICE1_CHANGED}"
                    echo "service-2 changed: ${env.SERVICE2_CHANGED}"
                }
            }
        }

        stage('Build service-1') {
            when {
                expression { env.SERVICE1_CHANGED == "true" }
            }
            steps {
                dir('service-1') {
                    bat 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build service-2') {
            when {
                expression { env.SERVICE2_CHANGED == "true" }
            }
            steps {
                dir('service-2') {
                    bat 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Deploy service-1 (Hot Deploy)') {
            when {
                allOf {
                    branch 'main'
                    expression { env.SERVICE1_CHANGED == "true" }
                }
            }
            steps {
                script {
                    deployService(
                        "service-1",
                        "service-1\\target\\service-1.war"
                    )
                }
            }
        }

        stage('Deploy service-2 (Hot Deploy)') {
            when {
                allOf {
                    branch 'main'
                    expression { env.SERVICE2_CHANGED == "true" }
                }
            }
            steps {
                script {
                    deployService(
                        "service-2",
                        "service-2\\target\\service-2.war"
                    )
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully"
        }
        failure {
            echo "Deployment failed"
        }
    }
}

def deployService(String appName, String warPath) {
    bat """
        echo Deploying ${appName}

        if not exist "%BACKUP_ROOT%\\${appName}" mkdir "%BACKUP_ROOT%\\${appName}"
        copy ${warPath} "%BACKUP_ROOT%\\${appName}\\${appName}_%BUILD_NUMBER%.war"

        if exist "%TOMCAT_HOME%\\webapps\\${appName}" (
            rmdir /S /Q "%TOMCAT_HOME%\\webapps\\${appName}"
        )
        if exist "%TOMCAT_HOME%\\webapps\\${appName}.war" (
            del "%TOMCAT_HOME%\\webapps\\${appName}.war"
        )

        copy ${warPath} "%TOMCAT_HOME%\\webapps\\${appName}.war"
    """
}
