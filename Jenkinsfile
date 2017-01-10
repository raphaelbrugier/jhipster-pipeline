#!groovy​

node {
    def nodeHome = tool name: 'node-7.1.0', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
    env.PATH = "${nodeHome}/bin:${env.PATH}"

    stage('check tools') {
        sh "node -v"
        sh "npm -v"
        sh "bower -v"
        sh "gulp -v"
        sh "aws --version"
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
            throw err
        } finally {
          step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
        }
    }

    stage('frontend tests') {
      try {
          sh "gulp test"
      } catch(err) {
          throw err
      } finally {
        step([$class: 'JUnitResultArchiver', testResults: '**/target/test-results/karma/TESTS-*.xml'])
      }
    }

    stage('packaging') {
        sh "./mvnw package -Pprod -DskipTests"
    }

    stage('publishing') {
        sh "aws s3 cp target/jhipster-0.0.1-SNAPSHOT.war  s3://jhipipeline/jhipster-0.0.1-SNAPSHOT.war"
    }

    stage('deploying') {
        sh "ssh -oStrictHostKeyChecking=no -i ~/.ssh/awsperso.pem ec2-user@$AWS_PROD_INSTANCE_IP 'aws s3 cp s3://jhipipeline/jhipster-0.0.1-SNAPSHOT.war .; chmod +x jhipster-0.0.1-SNAPSHOT.war'"
        sh """ssh -oStrictHostKeyChecking=no -i ~/.ssh/awsperso.pem ec2-user@$AWS_PROD_INSTANCE_IP 'ps -aux | grep -i java | grep -v grep | cut -d' ' -f2 | xargs kill -9'"""
        sh "ssh -oStrictHostKeyChecking=no -i ~/.ssh/awsperso.pem ec2-user@$AWS_PROD_INSTANCE_IP './jhipster-0.0.1-SNAPSHOT.war --spring.profiles.active=prod --spring.datasource.password=$AWS_PROD_DATABASE_PASSWORD --spring.datasource.username=jhipipe --spring.datasource.url=jdbc:mysql://$AWS_PROD_DATABASE_URL:3306/jhipipedbprod?useUnicode=true&characterEncoding=utf8&useSSL=false &'"
    }
}
