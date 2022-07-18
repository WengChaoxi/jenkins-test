#!/usr/bin/env groovy

pipeline {
    agent any

    parameters {
        booleanParam(name: 'CLEAN_WS',
            defaultValue: false,
            description: 'When checked, will clean workspace.')

        string(name: 'GORELEASER_VERSION',
            defaultValue: '1.10.2',
            description: 'The version of the goreleaser')

        string(name: 'PROVIDER_VERSION',
            defaultValue: '0.0.1',
            description: 'The version of the terraform provider')
    }

    stages {
        stage("clean") {
            steps {
                script {
                    if (params.CLEAN_WS) {
                        sh 'git tag -d `git tag`'
                        sh 'git reset HEAD --hard'
                        // cleanWs()
                    }
                }
            }
        }

        stage("github release") {
            environment {
                GITHUB_TOKEN = credentials('github-token')
                GPG_PRIVATE_KEY_FILE = credentials('gpg-private-key-file')
                GPG_FINGERPRINT = credentials("gpg-fingerprint")
            }
            steps {
                script {
                    assert params.GORELEASER_VERSION
                    assert params.PROVIDER_VERSION

                    sh "git tag v${params.PROVIDER_VERSION}"

                    withGo('Go 1.18') {
                        sh """
                            go install github.com/goreleaser/goreleaser@v${params.GORELEASER_VERSION}
                            gpg --import "$GPG_PRIVATE_KEY_FILE"
                            goreleaser release --rm-dist
                        """
                    }
                }
            }
        }
    }
}
