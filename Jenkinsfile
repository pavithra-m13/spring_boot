pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-21'
            args '--network devnet'
        }
    }
    environment {
       
        
        // Option 2: If using single username/password credential (comment out Option 1 and use this)
        NEXUS_CREDS = credentials('nexus-creds')
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
                 

                    // Option 2: Using single credential (uncomment if using NEXUS_CREDS)
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
                }
                
                // For SNAPSHOT versions, use snapshots repository
                sh 'mvn deploy --settings temp-settings.xml -DaltDeploymentRepository=nexus::default::http://172.18.0.2:8081/repository/maven-snapshots/'
                
                // Or for release versions (if you change version to 0.0.1 without -SNAPSHOT)
                // sh 'mvn deploy --settings temp-settings.xml -DaltDeploymentRepository=nexus::default::http://172.18.0.2:8081/repository/maven-releases-custom/'
            }
        }
    }
    post {
        always {
            sh 'rm -f temp-settings.xml'
        }
    }
}
