/* ─────────────────────────  Jenkinsfile  ───────────────────────── */
pipeline {
    /* Force the whole pipeline onto the agent(s) labelled “slavenode” */
    agent { label 'slavenode' }

    tools {
        maven 'Maven3'     // must match the name in Manage Jenkins → Tools
        jdk   'Java17'
    }

    environment {
        SONARQUBE_ENV = 'SonarQubeServer'
        NEXUS_URL     = 'http://34.173.82.220:8081/repository/maven-snapshots/'
        TOMCAT_URL    = 'http://34.55.221.192:8080/manager/text'
        WAR_FILE      = 'target/SampleWebApplication.war'
    }

    stages {
        /* ─── 1. Ensure we’re on the right machine  ─── */
        stage('Verify correct node') {
            steps {
                script {
                    if (env.NODE_NAME != 'slave-instance') {
                        error "💥  Expected to run on slave-instance, but got ${env.NODE_NAME}. Aborting."
                    }
                    echo "✅ Verified: running on ${env.NODE_NAME}"
                }
            }
        }

        /* ─── 2. Pull source  ─── */
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Navya89k/maven-web-application.git'
            }
        }

        /* ─── 3. Static analysis  ─── */
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        /* ─── 4. Build WAR  ─── */
        stage('Build WAR') {
            steps {
                sh 'mvn package'
            }
        }

        /* ─── 5. Upload to Nexus  ─── */
        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion:  'nexus3',
                    protocol:      'http',
                    nexusUrl:      '34.173.82.220:8081',
                    groupId:       'com.javacodegeeks',
                    version:       '1.0-SNAPSHOT',
                    repository:    'maven-snapshots',
                    credentialsId: 'nexus-creds-id',
                    artifacts: [[
                        artifactId: 'maven-web-app',
                        classifier: '',
                        file:       'target/SampleWebApplication.war',
                        type:       'war'
                    ]]
                )
            }
        }

        /* ─── 6. Deploy to Tomcat  ─── */
        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-cred-id',
                                          url: "${TOMCAT_URL}")],
                       contextPath: 'SampleWebApplication',
                       war: "${env.WAR_FILE}"
                echo "Deployment successfully completed"
            }
        }
    }
}
/* ───────────────────────────────────────────────────────────────── */
