#!/usr/bin/env groovy

pipeline {
    agent any

    parameters {
        booleanParam(name: 'CLEAN_WS',
            defaultValue: false,
            description: 'When checked, will clean workspace.')

        string(name: 'VERSION',
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
                GITHUB_TOKEN = "${env.GITHUB_TOKEN}"
                GPG_PRIVATE_KEY = credentials('gpg-private-key')
                GPG_FINGERPRINT = "admin <admin@wengcx.top>"
            }
            steps {
                sh "git tag v${VERSION}"
                withGo('Go 1.18') {
                    sh """
                        TMPDIR="\$(mktemp -d)"
                        echo "$GPG_PRIVATE_KEY" > "\$TMPDIR/secret.txt"
                        gpg --import "\$TMPDIR/secret.txt"
                        rm -f "\$TMPDIR/secret.txt"

                        # go install github.com/goreleaser/goreleaser@latest
                        goreleaser release --rm-dist
                    """
                }
            }
        }
    }
}
