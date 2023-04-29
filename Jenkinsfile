node{
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        }

    stage('Clone Git Repo'){
        git branch: 'main', url: 'https://github.com/ManjuNK/jenkins-maven-web-app-docker'
    }

    stage('Maven Build'){
        def mavenHome = tool name: "Maven-3.8.6", type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }

    stage('SonarQube Code Analysis') {
        withSonarQubeEnv('Sonar-Server-9.4') {
        sh "mvn sonar:sonar"
    }

    stage('Upload Artificate to Nexus Repo'){
nexusArtifactUploader artifacts: [[artifactId: '02-maven-web-app', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], credentialsId: 'Nexus-Credentails', groupId: 'in.manju', nexusUrl: '3.64.237.2:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'manju-snapshot-nexus-repo-docker', version: '2.0-SNAPSHOT'     
    }

    stage('Build Docker Image'){
        sh 'docker build -t manjunk/mavenwebapp .'
    }

    stage('Push Docker Image'){
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) 
        {
        	sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"

            sh 'docker push manjunk/mavenwebapp'
        }
    }
}
}