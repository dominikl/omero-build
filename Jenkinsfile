pipeline {
    agent {
        label 'testintegration'
    }

    environment {
        // Default credentials for testing on devspace
        MAVEN_SNAPSHOTS_REPO_URL = 'http://nexus:8081/nexus/repository/maven-internal/'
        MAVEN_USER = 'admin'
        MAVEN_PASSWORD = 'admin123'

        // Disable Gradle daemon
        GRADLE_OPTS = '-Dorg.gradle.daemon=false'
    }

    stages {
        stage('Versions') {
            steps {
                // build is in .gitignore so we can use it as a temp dir
                sh """
                    mkdir ${env.WORKSPACE}/build
                    cd ${env.WORKSPACE}/build && curl -sfL https://github.com/ome/build-infra/archive/master.tar.gz | tar -zxf -
                    export PATH=$PATH:${env.WORKSPACE}/build/build-infra-master/
                    cd ..
                    foreach-get-version-as-property >> version.properties
                """
                archiveArtifacts artifacts: 'version.properties'
            }
        }
        stage('Build') {
            steps {
                // Currently running on a build node with multiple jobs so incorrect jar may be cached
                // (Moving to Docker should fix this)
                sh 'gradle --init-script init-ci.gradle publishToMavenLocal --refresh-dependencies'
            }
        }
        stage('Deploy') {
            steps {
                sh 'gradle --init-script init-ci.gradle publish'
            }
        }
    }

    post {
        always {
            // Cleanup workspace
            deleteDir()
        }
    }
}
