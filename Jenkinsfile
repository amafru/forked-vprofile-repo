//Slack Notifications color mapping definitions. See further down for additional params
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline{

    agent any
    
    tools{
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    stages{
        //stage('Print Error message'){
        //    steps{
        //        sh 'generate a failure notification'
        //    }
        //}
        
        stage('Fetch Code'){
            steps{
                git branch: 'vp-rem', url: 'https://github.com/amafru/forked-vprofile-repo.git'
            }
        }
    stage('Build'){
        steps{
            sh 'mvn install -DskipTests'
        }

        post{
            success{
                echo 'Now Archiving the artefact...'
                archiveArtifacts artifacts: '**/target/*.war'
            }
        }
    }

    stage('UNIT TEST'){
        steps{
            sh 'mvn test'
        }
    }

    stage('CheckStyle Analysis'){
        steps{
            sh 'mvn checkstyle:checkstyle'
        }
    }

    stage('Sonar Analysis'){
        environment{
            scannerHome = tool 'sonar4.7'
        }
        steps{
            withSonarQubeEnv('sonar'){
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                -Dsonar.projectName=vprofile \
                -Dsonar.ProjectVersion=1.0 \
                -Dsonar.sources=src/ \
                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        //Applying a bespoke quality gate 
        stage("Quality Gate"){
            steps{
                timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload Artifact to Nexus"){
            steps{
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: '172.31.80.57:8081',
                groupId: 'DEV',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: 'Vprofile-Repo',
                credentialsId: 'nexuslogin',
                artifacts: [
                    [artifactId: 'vprofileapp',
                    classifier: '',
                    file: 'target/vprofile-v2.war',
                    type: 'war']
                    ]
                )
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkins-cicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}