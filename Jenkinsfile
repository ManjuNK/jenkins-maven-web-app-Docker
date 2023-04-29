node{

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

    stage('upload war to nexus'){
        steps{
                nexusArtifactUploader artifacts: [
                        [
                                artifactId: '01-maven-web-app',
                                classifier: '',
                                file: 'target/01-maven-web-app.war',
                                type: war
                        ]
                ],
                credentialsId: 'nexus3',
                groupId: 'in.manju',
                nexusUrl: '3.64.237.2:8081/',
                protocol: 'http',
                repository: 'manju-snapshot-nexus-repo-docker'
                version: '1.0-SNAPSHOT'
           }
       }

    stage('Build Image'){
        sh 'docker build -t manjunk/mavenwebapp .'
    }

    stage('Push Image'){
        withCredentials([string(credentialsId: 'DOCKER-CREDENTIALS', variable: 'docker-cred')]) {
            sh 'docker login -u manjunk -p ${docker-cred}'
        }
        sh 'docker push manjunk/mavenwebapp'
    }

    stage('Deploy App'){
        kubernetesDeploy(
            configs: 'maven-web-app-deploy.yml',
            kubeconfigId: 'Kube-Config'
        )
    }
}
