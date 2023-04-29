node{
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        }

    stage('Clone repo'){
        git branch: 'main', url: 'https://github.com/ManjuNK/jenkins-maven-web-app-docker'
    }

    stage('Maven Build'){
        def mavenHome = tool name: "Maven-3.8.6", type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }

    stage('SonarQube analysis') {
        withSonarQubeEnv('Sonar-Server-9.4') {
        sh "mvn sonar:sonar"
    }

    stage('Upload Artifcate'){
nexusArtifactUploader artifacts: [[artifactId: '02-maven-web-app', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], credentialsId: 'Nexus-Credentails', groupId: 'in.manju', nexusUrl: '3.64.237.2:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'manju-snapshot-repository', version: '2.0-SNAPSHOT'     
    }

    stage('Build Image'){
        sh 'docker build -t manjunk/mavenwebapp .'
    }

    stage('Push Image'){
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
        	sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"

        sh 'docker push manjunk/mavenwebapp'
        }

    stage('Deploy App'){
        kubernetesDeploy(
            configs: 'maven-web-app-deploy.yml',
            kubeconfigId: 'Kube-Config'
        )
    }
}
