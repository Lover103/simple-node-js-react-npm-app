pipeline {
    agent { docker 'node' }
    options {
        gitLabConnection('gitlab-bigdata')
        disableConcurrentBuilds()
        skipDefaultCheckout()
        timeout(time: 60, unit: 'MINUTES')
    }

    triggers{
        gitlab(
            triggerOnPush: true, 
            triggerOnMergeRequest: true,
            branchFilterType: 'All'
        )
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    echo "Prepare Stage"

                    checkout scm
                    updateGitlabCommitStatus name: 'build', state: 'pending'
                }
            }
        }

        stage('Compile And UnitTest') {
            steps {
                script {
                    echo "Compile the code"

                    try {
                        sh "node --version"
                        sh 'npm config set registry http://registry.npm.taobao.org/'
                        sh 'npm install'
                        sh 'npm run build'
                    } catch(Exception exception){
                        updateGitlabCommitStatus name: 'build', state: exception
                        throw exception;
                    } finally {
                        updateGitlabCommitStatus name: 'build', state: 'failed'
                    }

                    updateGitlabCommitStatus name: 'build', state: 'success'
                    updateGitlabCommitStatus name: 'Basic Quality Check', state: 'pending'
                }
            }
        }

        stage('Basic Quality Report') {
            steps {
                script {
                    echo "Basic quality report"
                }
            }
        }

        stage('Basic Quality Check') {
            steps {
                script {
                    echo "Check quality threshold"

                    try {
                        echo "Just skip check for demo, but should check when work"
                    } catch(Exception ex){
                        updateGitlabCommitStatus name: 'Basic Quality Check', state: 'failed'
                        throw ex;
                    } finally {

                    }
                    updateGitlabCommitStatus name: 'Basic Quality Check', state: 'success'
                }
            }
        }

        stage('SonarQube analysis') {
            steps {
                script {
                    updateGitlabCommitStatus name: 'SonarQube analysis', state: 'pending'
                }
            }
        }
    }

        stage('测试环境部署') {
            when { branch 'developer' }
            steps {
                script {
                  echo "测试环境部署"
                }
            }
        }

        stage('正式环境部署') {
            when { branch 'master' }
            steps {
                script {
                  echo "正式环境部署"
                }
            }
        }
    }

    post {
        always {
            // deleteDir()
            echo 'Test run completed'
            cucumber buildStatus: 'UNSTABLE', failedFeaturesNumber: 999, failedScenariosNumber: 999, failedStepsNumber: 3, fileIncludePattern: '**/*.json', skippedStepsNumber: 999
        }
        success {
            echo 'Successfully!'
            updateGitlabCommitStatus name: 'build', state: 'success'
            // mail(
            //     from: "quan.shi@zymobi.com",
            //     to: "quan.shi@zymobi.com",
            //     subject: "That build passed.",
            //     body: "Nothing to see here"
            // )
        }
        failure {
            echo 'Failed!'
            updateGitlabCommitStatus name: 'build', state: 'failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}