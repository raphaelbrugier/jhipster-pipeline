node {
    // uncomment these 2 lines and edit the name 'node-4.6.0' according to what you choose in configuration
    def nodeHome = tool name: 'node-7.1.0', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
    env.PATH = "${nodeHome}/bin:${env.PATH}"

    stage('check tools') {
        sh "node -v"
        sh "npm -v"
        sh "bower -v"
        sh "gulp -v"
    }

    stage('checkout') {
        checkout scm
    }

    stage('npm install') {
        sh "npm install"
    }

    stage('clean') {
        sh "./mvnw clean"
    }

    stage('backend tests') {
        try {
            sh "./mvnw test"
        } catch(err) {
            step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
            step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
            throw err
        }
    }

    stage('frontend tests') {
        sh "gulp test"
    }

    stage('packaging') {
        sh "./mvnw package -Pprod -DskipTests"
    }
}
