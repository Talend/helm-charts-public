#!/usr/bin/groovy

def slackChannel = 'dpe-build-notification'
def decodedJobName = env.JOB_NAME.replaceAll("%2F", "/")

pipeline {

    options {
        disableConcurrentBuilds()
        buildDiscarder logRotator(artifactDaysToKeepStr: '5', artifactNumToKeepStr: '5', numToKeepStr: '10')
        timeout(time: 10, unit: 'MINUTES')
    }

    agent {
        kubernetes {
            yamlFile 'builderPodTemplate.yaml'
            defaultContainer 'talend-tsbi-dev'
        }
    }

    environment {
        ARTIFACTORY_URL = 'https://artifactory.datapwn.com/artifactory'
        PUBLIC_REPO_URL = 'https://talend.github.io/helm-charts-public'
        GIT_USER = 'ci-build-dpe'
        GIT_EMAIL = 'ci-build-dpe@talend.com'
        GIT_CREDS = credentials('github-credentials')
    }

    parameters {
        string(name: 'CHART_NAME', defaultValue: '', description: 'Chart name + version to the internal chart to deploy (ie: dpe-remote-engine-client-1.2.0)')
        string(name: 'CHART_REPO', defaultValue: 'tlnd-helm-ce-dev', description: 'The repo to use')
        string(name: 'CHART_PUBLIC_FOLDER', defaultValue: 'engine', description: 'Chart root folder')
    }

    stages {
        stage('Initialisation') {
            steps {
                sh '''
                     #! /bin/bash
                     set +x

                     echo https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com > /tmp/.git-credentials
                     git config credential.helper 'store --file /tmp/.git-credentials\'
                     git config user.name "$GIT_USER"
                     git config user.email "$GIT_EMAIL"
                  '''
            }
        }

        stage('Download chart') {
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'artifactory-datapwn-credentials', passwordVariable: 'ARTIFACTORY_DOCKER_PASSWORD', usernameVariable: 'ARTIFACTORY_DOCKER_LOGIN')
                ]) {
                    sh '''
                        mkdir $CHART_PUBLIC_FOLDER/$CHART_NAME
                        cd $CHART_PUBLIC_FOLDER/$CHART_NAME
                        helm pull $ARTIFACTORY_URL/$CHART_REPO/$CHART_NAME.tgz --username $ARTIFACTORY_DOCKER_LOGIN --password $ARTIFACTORY_DOCKER_PASSWORD
                    '''
                }
            }
        }

        stage('Update chart repo index') {
            steps {
                sh '''
                    ls -l
                    cd ${params.CHART_PUBLIC_FOLDER}/${params.CHART_NAME}
                    helm repo index --merge ../index.yaml --url ${PUBLIC_REPO_URL}/${params.CHART_PUBLIC_FOLDER}/ .
                    mv -f index.yaml ../index.yaml
                '''
            }
        }

        stage('Push git') {
            when {
                branch 'master';
            }
            steps {
                sh '''
                    git commit -m "Added chart ${params.CHART_NAME}"
                    git push origin master
                '''
            }
        }
    }

    post {

        unstable {
            slackSend(color: 'warning', channel: "${slackChannel}", message: "UNSTABLE: `${decodedJobName}` #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        failure {
            slackSend(color: '#e81f3f', channel: "${slackChannel}", message: "FAILED: `${decodedJobName}` #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

    }
}
