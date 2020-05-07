# Template repo for new CI configuration (GitHub Branch Source plugin for Jenkins)

- You must have a `Jenkinsfile` in the root of the repo defining how your project is built/tested/deployed. Don't forget the last 3 lines which are needed for building by PR comment.
```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'gcc ...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
    
    triggers {
        issueCommentTrigger('launch jenkins')
    }
}
```
- Add a GitHub workflow that adds a link to the PR's jenkins page on the PR. See `.github/workflows/pr_build_link.yml`.
- In Settings / Webhooks add a webhook:
  - Payload URL: `https://jenkins-mch.cscs.ch/github-webhook/`
  - Content type: `application/json`
  - Enable SSL verification
  - Select these events to trigger the webhook: Issue comments, Pull requests
