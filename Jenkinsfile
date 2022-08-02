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
        string(name: 'RELEASE_VERSION',
            defaultValue: '',
            description: 'The version of the terraform provider. (MAJOR.MINOR.PATCH, e.g. 0.0.1). If the version is left empty, release stage will be skipped.',
            trim:true)
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }
        stage("Release") {
            when {
                expression {
                    params.RELEASE_VERSION != ''
                }
            }
            steps {
                terraformProviderRelease(releaseVersion: params.RELEASE_VERSION)
            }
        }
    }
}
