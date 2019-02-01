#!groovy
//
@NonCPS
void comment_issues() {
    def issue_pattern = "DEMO-\\d+"

    // Find all relevant commit ids
    currentBuild.changeSets.each {changeSet ->
        changeSet.each { commit ->
            String msg = commit.getMsg()
            msg.findAll(issue_pattern).each {
                // Actually post a comment
                id -> jiraComment body:msg, issueKey:id
            }
        }
    }
}

pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000 -p 5000:5000 -e HOME=.' 
        }
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test Feature Branch') {
            when {
                branch  'feature-*'
        }
            steps {
                 sh ' echo "test branch $BRANCH "'
                 sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                 sh './jenkins/scripts/kill.sh'
             }
       }
        stage('Run Tests') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Update Jira') {
            steps {
                comment_issues()
            }
        }
    }
}


