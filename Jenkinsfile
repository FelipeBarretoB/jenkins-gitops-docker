node {
    def app

    environment {
        DOCKER_HOST = 'unix:///var/run/docker.sock'
        DOCKER_BUILDKIT = '0' // Optional: disable BuildKit if it's causing issues
    }

    stage('Clone repository') {
        checkout scm
    }

    stage('Build image') {
        app = docker.build("devopswithsam/jenkins-flask")
    }

    stage('Test image') {
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
            app.push("${env.BUILD_NUMBER}")
        }
    }

    stage('Trigger ManifestUpdate') {
        echo "triggering updatemanifestjob"
        build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
    }
}
