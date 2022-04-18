def argocdServer = "argocd.10.38.16.13.nip.io"

podTemplate(containers: [
    containerTemplate(name: 'builder', image: 'golang:1.10.3', ttyEnabled: true, command: 'cat', args: ''),
    containerTemplate(name: 'docker', image: 'docker:17.09', ttyEnabled: true, command: 'cat', args: '' ),
    containerTemplate(name: 'argo-cd-tools', image: 'argoproj/argo-cd-tools:latest', ttyEnabled: true, command: 'cat', args: '', envVars:[envVar(key: 'GIT_SSH_COMMAND', value: 'ssh -o StrictHostKeyChecking=no')] ),
    containerTemplate(name: 'argo-cd-cli', image: 'argoproj/argocd-cli:v0.7.1', ttyEnabled: true, command: 'cat', args: '', envVars:[envVar(key: 'ARGOCD_SERVER', value: argocdServer)] ),
    ],
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
  ){

  node(POD_LABEL) {

    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
      }
      container('docker') {
          stage('Build Docker Image') {
            // Build new image
            sh "docker build -t ntnxdemo/argocd-demo:${env.GIT_COMMIT} ."
            // Publish new image
            sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker push ntnxdemo/argocd-demo:${env.GIT_COMMIT}"
          }
      }
    }

    stage('Deploy E2E') {
        environment {
          GIT_CREDS = credentials('git')
        }
        container('argo-cd-tools') {
          stage('Deploy E2E') {
            sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/ntnxdemo/argocd-demo-deploy.git"
            sh "git config --global user.email 'admin@no-reply.com'"

            dir("argocd-demo-deploy") {
              sh "cd ./e2e && kustomize edit set image ntnxdemo/argocd-demo:${env.GIT_COMMIT}"
              sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
            }
        }
      }
    }
  }
}