# Template repository for new CI configuration (GitHub Branch Source plugin for Jenkins)

Since the old jenkins plugin for PRs ([Jenkins Pull Request Builder plugin](https://plugins.jenkins.io/ghprb/)) is not maintained anymore, we decided to migrate to a new configuration for building the code of pull requests.

In this new configuration we are using the [Github Branch Source plugin](https://plugins.jenkins.io/github-branch-source/), which recognizes pull requests of repositories under the MeteoSwiss-APN GitHub organization (or their forks from authorized users) and automatically adds "jobs" on Jenkins for them. It is then possible to launch the build of such jobs by navigating to their pages (on Jenkins): `Jenkins - MeteoSwiss-APN GitHub - <repository> - Pull Requests - PR-<PR_ID>`.

Since navigating to the Jenkins page takes some time, we are using a GitHub Workflow script that adds a comment to a newly opened PR with the link to the relative job on Jenkins. From there, a logged user, can hit the "Build Now" button.

Unfortunately there is no way to avoid doing this when you want to build a new PR for the first time. However, after the first build, one can write the usual comment "launch jenkins" on the PR page on GitHub to re-trigger the build, thanks to [this plugin](https://github.com/jenkinsci/pipeline-github-plugin).

## Setup instructions

- You must have a `Jenkinsfile` in the root of the repo defining how your project is built/tested/deployed. Here is an example one. Make sure to include the last 3 lines which are needed for building (after the first build) by commenting "launch jenkins".
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
- Add a GitHub workflow (from your repository page click on "Actions", then "New Workflow", then "setup a workflow yourself") that adds a link to the Jenkins job page on the PR. Just copy the one in `.github/workflows/pr_build_link.yml` on this repository.
- From your repository page - Settings - Webhooks, add a webhook:
  - Payload URL: `https://jenkins-mch.cscs.ch/github-webhook/`
  - Content type: `application/json`
  - "Enable SSL verification"
  - "Let me select individual events.", then select these events to trigger the webhook: Issue comments, Pull requests
