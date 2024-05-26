pipeline{
    agent any
    environment{
        SONAR_HOME= tool "Sonar"
    }
    stages{
        stage("Clone Code from GitHub"){
            steps{
                git url: "https://github.com/syedusmanalishah/pos.git", branch: "master"
            }
        }
      stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Deploy using Docker Compose") {
            steps {
                script {
                    // Stop and remove any existing containers
                    sh "docker compose down"
                    
                    // Start new containers
                    sh "docker compose up -d"
                }
            }
        }
        stage("MongoDB Data Import") {
            steps {
                script {
                    // Ensure MongoDB container is up before running the import command
                    def mongoContainer = sh(script: "docker ps -q -f name=mongo", returnStdout: true).trim()
                    if (mongoContainer) {
                        sh "docker exec ${mongoContainer} mongoimport --db pos --collection items --file /data/sample_posts.json --jsonArray"
                    } else {
                        error "MongoDB container is not running"
                    }
                }
            }
        }
    }
}
