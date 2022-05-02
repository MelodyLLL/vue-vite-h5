

pipeline {
    environment {
        FOO = ''
    }
    agent any
    options {
        skipStagesAfterUnstable()
    }
    parameters {
        choice(name: 'DEPLOY_TO', choices: ['Build-Only', 'FAT', 'PROD']
                , description: 'Env to deploy: PROD: production, FAT: test, Build-Only:build code only')
        choice(name: 'RELEASE_MODEL', choices: ['Prefer-Offline', 'Reinstall-Node-Modules']
                    , description: 'Release model')
    }
    stages {
        stage('Show Changes') {
            steps {
                lastChanges since: 'LAST_SUCCESSFUL_BUILD', format:'SIDE', matching: 'LINE'
                echo "Changes: ${env.BUILD_URL}/last-changes/"
            }
        }
        stage('Build-Reinstall') {
            when {
                expression { params.RELEASE_MODEL == 'Reinstall-Node-Modules' }
            }
            steps {
                echo 'About to reinstall node modules...(use this to upgrade dependencies)'
                sh 'npm install --no-audit'
                sh 'npm run build'
            }
        }
        stage('Build') {
            when {
                expression { params.RELEASE_MODEL == 'Prefer-Offline' }
            }
            steps {
                echo 'Building..'
                sh 'npm install --prefer-offline --no-audit'
                sh 'npm run build'
            }
        }
        stage('Prepare Deploy') {
            steps {
                echo 'About to checkout deployment script...'
                sh 'pwd ; ls -l'
                sh "rm -fr ${env.WORKSPACE}/infra"
                dir("${env.WORKSPACE}/infra") {
                    checkout([$class: 'GitSCM', branches: [[name: 'master']],
                              userRemoteConfigs: [[url: 'https://jenkins-user:smnVwvYLJRE_2CXv3JRA@gitlab.minisobos.com/engineering/infra.git']]])
                    sh 'chmod +x ./ansible/front_deploy.sh ; chmod +x ./ansible/post_deploy.sh'
                }
            }
        }
        stage('Deploy FAT') {
            when {
                expression { params.DEPLOY_TO == 'FAT' }
            }
            steps {
                dir("${env.WORKSPACE}/infra/ansible") {
                    withEnv(["DEPLOY_TO=${params.DEPLOY_TO}"]) {
                        sh './front_deploy.sh'
                        echo 'deploy fat'
                    }
                }
            }
        }
        stage('Confirmation') {
            when {
                branch 'master'
                expression { params.DEPLOY_TO == 'PROD' }
            }
            steps {
                input('Confirm to deploy to production')
            }
        }
        stage('Deploy PROD') {
            when {
                branch 'master'
                expression { params.DEPLOY_TO == 'PROD' }
            }
            steps {
                dir("${env.WORKSPACE}/infra/ansible") {
                    withEnv(["DEPLOY_TO=${params.DEPLOY_TO}"]) {
                        sh './front_deploy.sh'
                        echo 'deploy prod'
                    }
                }
            }
        }
        stage('Smoke Test') {
            steps {
                echo 'Performing smoke test'
            }
        }
    }
    // post {
    //     success {
    //         dir("${env.WORKSPACE}/infra/ansible") {
    //             withEnv(['JENKINS_BUILD_STATUS=success']) {
    //                 wrap([$class:'BuildUser']) {
    //                     sh 'test -f ./post_deploy.sh && ./post_deploy.sh'
    //                 }
    //             }
    //         }
    //     }
    //     unsuccessful {
    //         dir("${env.WORKSPACE}/infra/ansible") {
    //             withEnv(['JENKINS_BUILD_STATUS=unsuccessful']) {
    //                 wrap([$class:'BuildUser']) {
    //                     sh 'test -f ./post_deploy.sh && ./post_deploy.sh'
    //                 }
    //             }
    //         }
    //     }
    // }
}

