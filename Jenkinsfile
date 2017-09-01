import groovy.json.JsonOutput
// This Jenkinsfile's main purpose is to show a real-world-ish example
// of what Pipeline config syntax actually looks like. 
pipeline {
    // Make sure that the tools we need are installed and on the path.
    agent{
	   label 'develop'
    }

    tools {
    // Symbol for tool type and then name of configured tool installation
        jdk "JDK8u121"
    }

    stages {
        stage('checkout'){
            steps {
                checkout scm
                notifyAtomist("BUILDING", "STARTED")
            }
        }
        stage('echo'){
            steps {
                echo 'hello from beta'
                sh 'df -h'
                sh 'uname -a'
                sh 'java -version'
                sh 'which java'
            }
        }
        // While there is only one stage here, you can specify as many stages as you like!
        stage("build") {
            steps {
                sh "docker build -t poc-microservice-a:test -f POCDockerfile ."
                sh "docker rmi poc-microservice-a:test"
            }
        }
    }
    post {
        success {
            notifyAtomist("SUCCESS")
            echo 'This will run only if success'
        }
        unstable {
            notifyAtomist("UNSTABLE")
            echo 'This will run only if unstable'
        }
        failure {
            notifyAtomist("FAILURE")
            echo 'This will run only if failed'
        }
        changed {
            notifyAtomist("CHANGED")
            echo 'This will run only if the state of the Pipeline has changed'
        }
    }
    
}

/*
 * Notify the Atomist services about the status of a build based from a
 * git repository.
 */
def notifyAtomist(buildStatus, buildPhase="FINALIZED",
                  endpoint="https://webhook.atomist.com/atomist/jenkins") {

    def payload = JsonOutput.toJson([
        name: env.JOB_NAME,
        duration: currentBuild.duration,
        build      : [
            number: env.BUILD_NUMBER,
            phase: buildPhase,
            status: buildStatus,
            full_url: env.BUILD_URL,
            scm: getSCMInformation()
        ]
    ])

    sh "curl --silent -XPOST -H 'Content-Type: application/json' -d '${payload}' ${endpoint}"
}

/*
 * Retrieve current SCM information from local checkout
 */
def getSCMInformation() {
    def gitRemoteUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
    def gitCommitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def gitBranchName = sh(returnStdout: true, script: 'git name-rev --always --name-only HEAD').trim().replace('remotes/origin/', '')

    return [
        url: gitRemoteUrl,
        branch: gitBranchName,
        commit: gitCommitSha
    ]
}