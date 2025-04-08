podTemplate(
  serviceAccount: 'jenkins-admin',
  containers: [
    containerTemplate(
      name: 'jnlp',
      image: 'msd117/jenkins-generic-agent:latest',
      ttyEnabled: true
    )
  ],
  envVars: [
    containerEnvVar(key: 'CALIBRE_VERSION', value: '1.0.0'),
    containerEnvVar(key: 'JELLYFIN_VERSION', value: '10.10.6')
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

    stage('Verify') {
      container('jnlp') {
        withCredentials(
          [
            string(credentialsId: 'server1-domain', variable: 'DOMAIN'),
            string(credentialsId: 'nfs1-server', variable: 'NFS_SERVER'),
            string(credentialsId: 'calibre-password', variable: 'CALIBRE_PASSWORD')
          ]
        ) {
          // Run the deploy script dry-run mode to validate the resources
          sh './tooling/deploy -d'
        }
      }
    }

    stage('Deploy') {
      container('jnlp') {
        withCredentials(
          [
            string(credentialsId: 'server1-domain', variable: 'DOMAIN'),
            string(credentialsId: 'nfs1-server', variable: 'NFS_SERVER'),
            string(credentialsId: 'calibre-password', variable: 'CALIBRE_PASSWORD')
          ]
        ) {
          sh './tooling/deploy'
        }
      }
    }
  }
}
