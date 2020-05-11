# Template repository for new CI configuration (GitHub Branch Source plugin for Jenkins)

Since the old jenkins plugin for PRs ([Jenkins Pull Request Builder plugin](https://plugins.jenkins.io/ghprb/)) is not maintained anymore, we decided to migrate to a new configuration for building the code of pull requests.

In this new configuration we are using the [Github Branch Source plugin](https://plugins.jenkins.io/github-branch-source/), which recognizes pull requests of repositories under the MeteoSwiss-APN GitHub organization (or their forks from authorized users) and automatically adds "jobs" on Jenkins for them. It is then possible to launch the build of such jobs by navigating to their page (on Jenkins): `Jenkins - MeteoSwiss-APN GitHub - <repository> - Pull Requests - PR-<PR_ID>`.

Since navigating to the Jenkins page takes some time, we are using a GitHub Workflow script that adds a comment to a newly opened PR with the link to the relative job on Jenkins. From there, a logged user, can hit the "Build Now" button.

Unfortunately there is no way to avoid doing this when you want to build a new PR for the first time. However, after the first build, one can write the usual comment "launch jenkins" on the PR page on GitHub to re-trigger the build.

## Setup instructions

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
