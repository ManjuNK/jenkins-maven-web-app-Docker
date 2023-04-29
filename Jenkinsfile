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

    stage('Upload Artifcate'){
nexusArtifactUploader artifacts: [[artifactId: '02-maven-web-app', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], credentialsId: 'Nexus-Credentails', groupId: 'in.manju', nexusUrl: '3.64.237.2:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'manju-snapshot-repository', version: '2.0-SNAPSHOT'     
    }

    stage('Build Image'){
        sh 'docker build -t manjunk/mavenwebapp .'
    }

    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "manjunk/mavenwebapp"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            //sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }

    stage('Deploy App'){
        kubernetesDeploy(
            configs: 'maven-web-app-deploy.yml',
            kubeconfigId: 'Kube-Config'
        )
    }
}
}