#!/usr/bin/groovy
@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def flow = new io.fabric8.Fabric8Commands()
clientsTemplate{
    dockerNode{
        checkout scm
        def v = getNewVersion {}
        dir('server'){
            container('docker'){
                sh "docker build -t fabric8/keycloak:${v} ."
                sh "docker push fabric8/keycloak:${v}"
            }
        }

        dir('server-postgres'){
            container('clients'){
                def project = 'fabric8io/keycloak'

                sh 'chmod 600 /root/.ssh-git/ssh-key'
                sh 'chmod 600 /root/.ssh-git/ssh-key.pub'
                sh 'chmod 700 /root/.ssh-git'
                
                sh "git remote set-url origin git@github.com:${project}.git"
                sh "git config --global user.email fabric8-admin@googlegroups.com"
                sh "git config --global user.name fabric8-release"

                def uid = UUID.randomUUID().toString()
                sh "git checkout -b versionUpdate${uid}"

                sh "sed -i -r 's/FROM.*/FROM fabric8\\/keycloak:${v}/g' Dockerfile"

                sh 'git add Dockerfile'

                def githubToken = flow.getGitHubToken()
                def message = "Update keycloak base image version ${v}"

                sh "git commit -m \"${message}\""
                sh "git push origin versionUpdate${uid}"

                id = flow.createPullRequest(message, project, "versionUpdate${uid}")

                sleep 5 // give a bit of time for GitHub to get itself in order after the new PR
                
                flow.mergePR(project, id)
                
            }
            
            container('docker'){
                sh "docker build -t fabric8/keycloak-postgres:${v} ."
                sh "docker push fabric8/keycloak-postgres:${v}"
            }
        }

        updateDownstreamDependencies(v)
    }
}

def updateDownstreamDependencies(v) {
  pushPomPropertyChangePR {
    propertyName = 'keycloak.version'
    projects = [
            'fabric8io/fabric8-platform'
    ]
    version = v
  }
}