pipeline {
    agent {
    docker {
        image 'maven:3.9.6-eclipse-temurin-21'
    }
    }

        

    environment {
        // Fetch Nexus credentials using Jenkins Credentials Binding
        NEXUS_USERNAME = credentials('nexus-creds') // Jenkins credentials ID
        NEXUS_PASSWORD = credentials('nexus-creds')
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

        stage('Deploy to Nexus') {
            steps {
                // Use inline Maven settings with env vars instead of a file
                writeFile file: 'temp-settings.xml', text: """
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus</id>
      <username>${env.NEXUS_USERNAME}</username>
      <password>${env.NEXUS_PASSWORD}</password>
    </server>
  </servers>
</settings>
                """

                sh 'mvn deploy --settings temp-settings.xml'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up temporary files...'
            sh 'rm -f temp-settings.xml'
        }
    }
}
