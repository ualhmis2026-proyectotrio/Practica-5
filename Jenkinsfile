pipeline {
  agent any

  tools {
    maven "maven default"
  }

  stages {
    
    stage('Git fetch') {
      steps {
        git branch: 'main', url: 'https://github.com/ualhmis2026-proyectotrio/Practica-5'
      }
    }
    
    stage ('Analysis') {
      steps {
        // Warnings next generation plugin required
       withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
          // Ejecutamos 'site' pasándole la API Key a Maven
          sh 'mvn site -DnvdApiKey=$NVD_API_KEY'
       }
      }
      post {
        success {
          dependencyCheckPublisher pattern: '**/target/site/dependency-check-report.xml'
          recordIssues enabledForFailure: true, tool: checkStyle()
          recordIssues enabledForFailure: true, tool: pmdParser()
          recordIssues enabledForFailure: true, tool: cpd()
          recordIssues enabledForFailure: true, tool: findBugs()
          recordIssues enabledForFailure: true, tool: spotBugs()
        }
      }
    }
  
    stage ('Documentation') {
      steps {
        sh "mvn javadoc:javadoc javadoc:aggregate"
      }
      post{
        always {
          step $class: 'JavadocArchiver', javadocDir: 'docs', keepAll: false
          publishHTML(target: [reportName: 'Maven Site', reportDir: 'target/site', reportFiles: 'index.html', keepAll: false])
        }
      }
    }
    
    stage('SonarQube Analysis') {
      steps {
        // Conecta con el servidor que has configurado
        withSonarQubeEnv('servidor_sonarqube') {
          // Extrae el token de forma segura
          withCredentials([string(credentialsId: 'sonar_server', variable: 'SONAR_LOGIN_TOKEN')]) {
            // Lanza el análisis a tu servidor en Azure
            sh 'mvn clean verify sonar:sonar -Dsonar.login=$SONAR_LOGIN_TOKEN'
          }
        }
      }
    }
    
    stage('Compile, Test, Package') {
      steps {
        // Ejecuta clean y package (incluye compile y test)
        sh "mvn clean package"
      }
      post {
        success {
          junit '**/target/surefire-reports/TEST-*.xml'
          archiveArtifacts '**/target/*.jar'
          jacoco( 
            execPattern: '**/target/jacoco.exec',
            classPattern: '**/target/classes',
            sourcePattern: '**/src/',
            exclusionPattern: '**/test/'
          )
          publishCoverage adapters: [jacocoAdapter('**/target/site/jacoco/jacoco.xml')]
        }
      }
    }
  }
}