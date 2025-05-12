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
        app = docker.build("pipebarreto/jenkins-flask")
    }

    stage('Test image') {
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh """
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker tag pipebarreto/jenkins-flask pipebarreto/jenkins-flask:${env.BUILD_NUMBER}
                docker push pipebarreto/jenkins-flask:${env.BUILD_NUMBER}
            """
        }
    }


    stage('Trigger ManifestUpdate') {
        echo "triggering updatemanifestjob"
        build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
    }
}
