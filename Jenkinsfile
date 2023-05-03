pipeline {
    agent {
        kubernetes {
            label 'docker-builder'
            defaultContainer 'node'
            yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins
  containers:
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
"""
    }
}
    options {
        skipStagesAfterUnstable()
    }
    environment {
        customImage = ''
    }

    stages {

        stage ('Build') {
            steps {
                container('docker') {
                    sh '''
                        docker image build . -f Dockerfile --tag hrbr.dev.castera.us/sfn/influxdb:1.${BUILD_ID}
                    '''
                }
            }
        }
        
        stage('Publish') {
            steps{
                container('docker') {
                    script {
                        docker.withRegistry( 'https://hrbr.dev.castera.us', 'harbor' ) {
                            sh '''
                                docker push hrbr.dev.castera.us/sfn/influxdb:1.${BUILD_ID}
                            '''
                        }
                    }
                }
            }
        }
    }
}