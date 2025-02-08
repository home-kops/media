podTemplate(
  serviceAccount: 'jenkins-admin',
  containers: [
    containerTemplate(
      name: 'jnlp',
      image: 'msd117/jenkins-generic-agent',
      ttyEnabled: true
    )
  ],
  namespace: 'devops-tools'
) {
  node(POD_LABEL) {
    stage('Clone') {
      container('jnlp') {
        git branch: 'main',
          credentialsId: 'github-home-kops-token',
          url: 'https://github.com/home-kops/media.git'
      }
    }

    stage('Deploy common media files') {
      container('jnlp') {
        sh 'kubectl apply -f ./common'
      }
    }

    stage('Verify') {
      container('jnlp') {
        withCredentials(
          [
            string(credentialsId: 'nfs1-server', variable: 'NFS_SERVER'),
            string(credentialsId: 'server1-domain', variable: 'DOMAIN')
          ]
        ) {
          withCredentials([string(credentialsId: 'calibre-password', variable: 'CALIBRE_PASSWORD')]) {
            dir('calibre') {
              sh '''
                envsubst < 00-secret.yaml | kubectl apply -f - --dry-run=server && \
                envsubst < 01-deployment.yaml | kubectl apply -f - --dry-run=server && \
                kubectl apply -f 02-service.yaml --dry-run=server && \
                envsubst < 03-ingress-route.yaml | kubectl apply -f - --dry-run=server
              '''
            }
          }

          dir('jellyfin') {
            sh '''
              envsubst < 00-deployment.yaml | kubectl apply -f - --dry-run=server
              kubectl apply -f 01-service.yaml --dry-run=server
              envsubst < 02-ingress-route.yaml | kubectl apply -f - --dry-run=server
              envsubst < 03-cronjob.yaml | kubectl apply -f - --dry-run=server
            '''
          }

          dir('jellyseerr') {
            sh '''
              envsubst < 00-deployment.yaml | kubectl apply -f - --dry-run=server
              kubectl apply -f 01-service.yaml --dry-run=server
              envsubst < 02-ingress-route.yaml | kubectl apply -f - --dry-run=server
            '''
          }
        }
      }
    }

    stage('Deploy') {
      container('jnlp') {
        withCredentials(
          [
            string(credentialsId: 'nfs1-server', variable: 'NFS_SERVER'),
            string(credentialsId: 'server1-domain', variable: 'DOMAIN')
          ]
        ) {
          withCredentials([string(credentialsId: 'calibre-password', variable: 'CALIBRE_PASSWORD')]) {
            dir('calibre') {
              sh '''
                envsubst < 00-secret.yaml | kubectl apply -f - && \
                envsubst < 01-deployment.yaml | kubectl apply -f - && \
                kubectl apply -f 02-service.yaml && \
                envsubst < 03-ingress-route.yaml | kubectl apply -f -
              '''
            }
          }

          dir('jellyfin') {
            sh '''
              envsubst < 00-deployment.yaml | kubectl apply -f -
              kubectl apply -f 01-service.yaml
              envsubst < 02-ingress-route.yaml | kubectl apply -f -
              envsubst < 03-cronjob.yaml | kubectl apply -f -
            '''
          }

          dir('jellyseerr') {
            sh '''
              envsubst < 00-deployment.yaml | kubectl apply -f -
              kubectl apply -f 01-service.yaml
              envsubst < 02-ingress-route.yaml | kubectl apply -f -
            '''
          }
        }
      }
    }
  }
}
