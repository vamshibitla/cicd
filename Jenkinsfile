pipeline {
    agent any

    tools {
        maven 'maven3'    // ensure a Maven tool with this name exists in Jenkins global config
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning GIT HUB Repo'
                git branch: 'master', url: 'https://github.com/vamshibitla/cicd.git'
            }
        }

        stage('Build Artifact') {
            steps {
                echo 'Build Artifact'
                sh '''
                  which mvn || true
                  mvn -v || true
                  mvn clean install -DskipTests=true
                '''
            }
        }

        stage('Sonar') {
            steps {
                echo 'Scanning project with SonarQube'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                      echo "Running Sonar scan..."
                      mvn sonar:sonar \
                        -Dsonar.host.url=http://18.212.199.8:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Nexus Deploy') {
            steps {
                echo "Deploying artifact to Nexus..."

                withCredentials([usernamePassword(
                    credentialsId: 'nexus-cred',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    script {
                        // Create a temporary settings.xml with server credentials (safer than CLI args)
                        def settingsPath = "${env.WORKSPACE}/temp-settings.xml"
                        writeFile file: settingsPath, text: """<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                        https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus-repo</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>"""

                        // Run mvn using the temporary settings file. Use -X for debug if needed.
                        sh """
                          echo "Using settings: ${settingsPath}"
                          mvn -s ${settingsPath} -X clean deploy -DskipTests=true \\
                            -DaltDeploymentRepository=nexus-repo::default::http://18.212.199.8:8081/repository/nexus-repo/
                        """
                    }
                }
            }
        }
    } // stages
}
