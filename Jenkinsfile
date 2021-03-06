job('po-tests-pr') {
    concurrentBuild()

    parameters {
        stringParam('sha1')
    }

    scm {
        git {
            remote {
                github('coreos/prometheus-operator')
                refspec('+refs/pull/*:refs/remotes/origin/pr/*')
            }
            branch('${sha1}')
        }
    }

    wrappers {
        credentialsBinding {
            amazonWebServicesCredentialsBinding{
                accessKeyVariable('AWS_ACCESS_KEY_ID')
                secretKeyVariable('AWS_SECRET_ACCESS_KEY')
                credentialsId('Jenkins-Monitoring-AWS-User')
            }
            usernamePassword('QUAY_ROBOT_USERNAME', 'QUAY_ROBOT_SECRET', 'quay_robot')
        }
    }

    triggers {
        githubPullRequest {
            useGitHubHooks()
            orgWhitelist(['coreos-inc'])
            allowMembersOfWhitelistedOrgsAsAdmin()
            triggerPhrase('test this please|please test this')
            extensions {
                commitStatus {
                    context('prometheus-operator-tests')
                    triggeredStatus('Tests triggered')
                    startedStatus('Tests started')
                    completedStatus('SUCCESS', 'Success')
                    completedStatus('FAILURE', 'Failure')
                    completedStatus('PENDING', 'Pending')
                    completedStatus('ERROR', 'Error')
                }
            }
        }
    }

    steps {
        shell('docker run -v $PWD:/go/src/github.com/coreos/prometheus-operator -w /go/src/github.com/coreos/prometheus-operator/ golang make generate')
        shell('git diff --exit-code')
    }

    steps {
        shell('docker run --rm -v $PWD:/go/src/github.com/coreos/prometheus-operator -w /go/src/github.com/coreos/prometheus-operator golang make test')
    }

    steps {
        shell('./scripts/jenkins/run-e2e-tests.sh')
    }

    publishers {
        postBuildScripts {
            steps {
                shell('./scripts/jenkins/post-e2e-tests.sh')
            }
            onlyIfBuildSucceeds(false)
            onlyIfBuildFails(false)
        }
    }
}

job('po-tests-master') {
    concurrentBuild()

    scm {
        git {
            remote {
                github('coreos/prometheus-operator')
            }
            branch('master')
        }
    }

    wrappers {
        credentialsBinding {
            amazonWebServicesCredentialsBinding{
                accessKeyVariable('AWS_ACCESS_KEY_ID')
                secretKeyVariable('AWS_SECRET_ACCESS_KEY')
                credentialsId('Jenkins-Monitoring-AWS-User')
            }
            usernamePassword('QUAY_ROBOT_USERNAME', 'QUAY_ROBOT_SECRET', 'quay_robot')
        }
    }

    triggers {
        githubPush()
        gitHubPushTrigger()
        cron('@daily')
        pollSCM{scmpoll_spec('')}
    }

    steps {
        shell('docker run -v $PWD:/go/src/github.com/coreos/prometheus-operator -w /go/src/github.com/coreos/prometheus-operator/ golang make generate')
        shell('git diff --exit-code')
    }

    steps {
        shell('docker run --rm -v $PWD:/go/src/github.com/coreos/prometheus-operator -w /go/src/github.com/coreos/prometheus-operator golang make test')
    }

    steps {
        shell('./scripts/jenkins/run-e2e-tests.sh')
    }

    publishers {
        postBuildScripts {
            steps {
                shell('docker tag quay.io/coreos/prometheus-operator-dev:$BUILD_ID quay.io/coreos/prometheus-operator:master')
                shell('docker push quay.io/coreos/prometheus-operator:master')
            }
            onlyIfBuildSucceeds(true)
        }
        postBuildScripts {
            steps {
                shell('./scripts/jenkins/post-e2e-tests.sh')
            }
            onlyIfBuildSucceeds(false)
            onlyIfBuildFails(false)
        }
    }
}
