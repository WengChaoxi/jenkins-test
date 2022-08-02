#!/usr/bin/env groovy

def terraformProviderBuild(Map params = [:]) {
    def buildVersion = params.get('buildVersion')
    assert buildVersion
    def goVersion = params.get('goVersion', '1.18.4')
    def goreleaserVersion = params.get('goreleaserVersion', '1.10.2')

    withGo(goVersion) {
        withGoReleaser(goreleaserVersion) {
            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN'),
                file(credentialsId: 'gpg-private-key-file', variable: 'GPG_PRIVATE_KEY_FILE'), 
                string(credentialsId: 'gpg-fingerprint', variable: 'GPG_FINGERPRINT')]) {
                sh """
                    git tag v"${buildVersion}"
                    test -z `git tag -l "v${buildVersion}"` && git tag "v${buildVersion}"
                    gpg --import "${env.GPG_PRIVATE_KEY_FILE}"
                    goreleaser build --rm-dist
                """
            }
        }
    }
}

pipeline {
    agent any
    parameters {
        string(name: 'VERSION',
            defaultValue: '0.0.1',
            description: 'The version of the terraform provider (e.g. `0.0.1`).',
            trim:true)
        booleanParam(name: 'RELEASE',
            defaultValue: false,
            description: 'When checked, will auto release.')
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }
        stage("Build") {
            steps {
                script {
                    assert params.VERSION
                    terraformProviderBuild(goVersion: '1.18.4', buildVersion: params.VERSION)
                }
            }
        }
        stage("Release") {
            steps {
                script {
                    assert params.VERSION
                    terraformProviderRelease(goVersion: '1.18.4', releaseVersion: params.VERSION)
                }
            }
        }
    }
}
