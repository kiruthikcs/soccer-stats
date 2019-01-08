#!groovy​

// FULL_BUILD -> true/false build parameter to define if we need to run the entire stack for lab purpose only
final FULL_BUILD = params.FULL_BUILD
// HOST_PROVISION -> server to run ansible based on provision/inventory.ini
final HOST_PROVISION = params.HOST_PROVISION

final GIT_URL = 'https://github.com/Djrohith/soccer-stats.git'
final NEXUS_URL = '34.221.40.216:8081'

stage('Build') {
    node {
        git GIT_URL
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            if(FULL_BUILD) {
                mvnHome = tool 'M3'
                sh "echo 'inside build'"
                
                 if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
                
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Unit Tests') {   
        node {
            withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                sh "mvn -B clean test"
                stash name: "unit_tests", includes: "target/surefire-reports/**"
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Integration Tests') {
        node {
            withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                sh "mvn -B clean verify -Dsurefire.skip=true"
                stash name: 'it_tests', includes: 'target/failsafe-reports/**'
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Static Analysis') {
        node {
            withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                withSonarQubeEnv('sonar'){
                    unstash 'it_tests'
                    unstash 'unit_tests'
                    sh 'mvn sonar:sonar -DskipTests'
                }
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Approval') {
        timeout(time:3, unit:'DAYS') {
            input 'Do I have your approval for deployment?'
        }
    }
}


if(FULL_BUILD) {
    stage('Artifact Upload') {
        node {
            unstash 'artifact'

            def pom = readMavenPom file: 'pom.xml'
            def file = "${pom.artifactId}-${pom.version}"
            def jar = "target/${file}.war"

            sh "cp pom.xml ${file}.pom"
            
            nexusArtifactUploader artifacts: [[artifactId: 'soccer-stats', classifier: '', file: 'target/soccer-stats.0.0.2.war', type: 'war']], credentialsId: '92a0b40b-83c4-4a1f-a901-a5859bbcb4a4', groupId: 'br.com.meetup.ansible', nexusUrl: '34.221.40.216:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'demo-snap', version: '0.0.2'

             
        }
    }
}

/**
stage('Deploy') {
    node {
        def pom = readMavenPom file: "pom.xml"
        def repoPath =  "${pom.groupId}".replace(".", "/") + 
                        "/${pom.artifactId}"

        def version = pom.version

        if(!FULL_BUILD) { //takes the last version from repo
            sh "curl -o metadata.xml -s http://${NEXUS_URL}/repository/ansible-meetup/${repoPath}/maven-metadata.xml"
            version = sh script: 'xmllint metadata.xml --xpath "string(//latest)"',
                         returnStdout: true
        }
        def artifactUrl = "http://${NEXUS_URL}/repository/ansible-meetup/${repoPath}/${version}/${pom.artifactId}-${version}.war"

        withEnv(["ARTIFACT_URL=${artifactUrl}", "APP_NAME=${pom.artifactId}"]) {
            echo "The URL is ${env.ARTIFACT_URL} and the app name is ${env.APP_NAME}"

            // install galaxy roles
            sh "ansible-galaxy install -vvv -r provision/requirements.yml -p provision/roles/"        

            ansiblePlaybook colorized: true, 
            credentialsId: 'ssh-jenkins',
            limit: "${HOST_PROVISION}",
            installation: 'ansible',
            inventory: 'provision/inventory.ini', 
            playbook: 'provision/playbook.yml', 
            sudo: true,
            sudoUser: 'jenkins'
        }
    } **/
}
