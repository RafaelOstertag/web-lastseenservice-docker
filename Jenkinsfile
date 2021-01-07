pipeline {
    agent {
        label "arm64&&docker"
    }

    triggers {
        pollSCM '@hourly'
    }

    options {
        ansiColor('xterm')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '15')
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        string defaultValue: 'none', description: 'Version of Artifact in Nexus', name: 'VERSION', trim: true
        booleanParam defaultValue: false, description: 'Deploy to Kubernetes', name: 'DEPLOY'
    }

    stages {
        stage('Build & Push Docker Image') {
            when {
                allOf {
                    expression { return params.VERSION != 'none' }
                    expression { return params.VERSION != '' }
                }
            }

            steps {
                sh "docker build --build-arg 'VERSION=${params.VERSION}' -t rafaelostertag/lastseen-service:${params.VERSION} docker"
                withCredentials([usernamePassword(credentialsId: '750504ce-6f4f-4252-9b2b-5814bd561430', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh 'docker login --username "$USERNAME" --password "$PASSWORD"'
                    sh "docker push rafaelostertag/lastseen-service:${params.VERSION}"
                }
            }
        }

        stage('Trigger deploy') {
            when {
                expression { return params.DEPLOY }
                allOf {
                    expression { return params.VERSION != 'none' }
                    expression { return params.VERSION != '' }
                }
            }

            steps {
                build wait: false, job: '../Helm/lastseenservice', parameters: [string(name: 'VERSION', value: params.VERSION)]
            }
        }
    }

    post {
        always {
            mail to: "rafi@guengel.ch",
                    subject: "${JOB_NAME} (${env.BUILD_DISPLAY_NAME}) -- ${currentBuild.currentResult}",
                    body: "Refer to ${currentBuild.absoluteUrl}"
        }
    }
}
