#!/usr/bin/env groovy

pipeline {
    agent any
    parameters {
        string(name: 'RELEASE_VERSION',
            defaultValue: '',
            description: 'The version of the terraform provider. (MAJOR.MINOR.PATCH, e.g. 0.0.1). If the version is left empty, release stage will be skipped.',
            trim:true)
    }
    options {
        skipDefaultCheckout()
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }
        stage("Build") {
            when {
                expression {
                    params.RELEASE_VERSION == ''
                }
            }
            steps {
                goreleaser goVersion: '1.18.4', snapshot: true
                archiveArtifacts artifacts: 'dist/*.zip', fingerprint: true
            }
        }
        stage("Release") {
            when {
                expression {
                    params.RELEASE_VERSION != ''
                }
            }
            steps {
                goreleaser goVersion: '1.18.4', releaseVersion: params.RELEASE_VERSION
            }
        }
    }
}

