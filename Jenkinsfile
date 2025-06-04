pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-21'
            args '--network devnet'
        }
    }
    environment {
        NEXUS_CREDS = credentials('nexus-creds') // Jenkins credentials with ID = nexus-creds
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/pavithra-m13/spring_boot.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Generate settings.xml with Nexus credentials
                    writeFile file: 'temp-settings.xml', text: """
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus</id>
      <username>${env.NEXUS_CREDS_USR}</username>
      <password>${env.NEXUS_CREDS_PSW}</password>
    </server>
  </servers>
</settings>
"""
                    // Use Maven deploy with proper repo
                    def projectVersion = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()

                    if (projectVersion.endsWith("-SNAPSHOT")) {
                        echo "Detected SNAPSHOT version: Deploying to snapshot repo"
                        sh 'mvn deploy --settings temp-settings.xml -DaltDeploymentRepository=nexus::default::http://nexus-container:8081/repository/maven-snapshots/'
                    } else {
                        echo "Detected RELEASE version: Deploying to release repo"
                        sh 'mvn deploy --settings temp-settings.xml -DaltDeploymentRepository=nexus::default::http://nexus-container:8081/repository/maven-releases-custom/'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'rm -f temp-settings.xml'
        }
    }
}
