#!groovy
//
//

@NonCPS
void comment_issues() {
    // Look for the following pattern within Git commits for Jira Ticket IDs
    def issue_pattern = "DEMO-\\d+"

    // FindAll instances of issue_pattern in the git commit message so we
    // can create Jira tickets using them as the IssueKey
    currentBuild.changeSets.each {changeSet ->
        changeSet.each { commit ->
            String msg = commit.getMsg()
            msg.findAll(issue_pattern).each {
                // Add a comment to Jira using the commit message
                // and Issue key extracted from the commit
                id -> jiraAddComment site: 'jira', comment:"$msg", idOrKey: id
            }
        }
    }
}

pipeline {
    agent {
        // Run these stages within a Docker container
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

        stage('Testing Feature Branch') {
            // This stage will only be executed on Git branches matching
            // the branch name
            when {
                branch  'feature-*'
        }
            steps {
                 sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                 sh './jenkins/scripts/kill.sh'
             }
       }
        stage('Running Tests') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Updating Jira') {
            steps {
            comment_issues()
        }
    }
}
}

