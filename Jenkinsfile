#!groovy

def cleanup = { ->
    // cleanup can't be a stage because it'll throw off the stage-view on the job's main page
    if (currentBuild.result != 'SUCCESS') {
        withCredentials([usernamePassword(
        credentialsId: '386d3445-b855-40e4-999a-dc5801336a69',
        passwordVariable: 'GAUNTLET_PASSWORD',
        usernameVariable: 'GAUNTLET_USERNAME'
        )]) {
            sh "curl -i --user ${GAUNTLET_USERNAME}:${GAUNTLET_PASSWORD} -X PUT 'https://gauntlet.wbx2.com/api/queues/react-ciscospark/master?componentTestStatus=failure&commitId=${GIT_COMMIT}'"
        }
    }
}

ansiColor('xterm') {
    timestamps {
        timeout(10) {
            node('NODE_JS_BUILDER') {
                
                def GIT_COMMIT

                try {
                    
                    stage('checkout') {
                        checkout scm

                        //sh 'git config user.email spark-js-sdk.gen@cisco.com'
                        //sh 'git config user.name Jenkins'

                        GIT_COMMIT = sh script: 'git rev-parse HEAD | tr -d "\n"', returnStdout: true
                        
                        sshagent(['6c8a75fb-5e5f-4803-9b6d-1933a3111a34']) {
                            sh 'git fetch upstream'
                            sh 'git checkout upstream/master'
                        }

                        try {
                            sh "git merge --ff ${GIT_COMMIT}"
                        }
                        catch (err) {
                            currentBuild.description = 'not possible to fast forward'
                            throw err;
                        }
                    }
                    
                    stage('Build'){
                        echo "RESULT: ${currentBuild.result}"
                         sh '''#!/bin/bash -ex
                         source ~/.nvm/nvm.sh
                         nvm use v6
                         npm install
                         npm run build
                        '''
                    }
                    
                    //archive 'dist/**/*'
                    archive 'packages/node_modules/@ciscospark/widget-message-meet/dist/**/*'

                    echo "RESULT: ${currentBuild.result}"

                    if (current.Build.result == 'SUCCESS'){
                        stage('Push to github'){
                            sshagent(['6c8a75fb-5e5f-4803-9b6d-1933a3111a34']) {
                           //     sh "git push upstream HEAD:master"
                            }
                        }

                        stage('Publish to CDN'){
                            // Need to create job(s) to publish to CDN
                            // If using a single job, job will need to be modified to copy artifacts
                            // in different locations then upload to the correct folder structre on CDN
                            // cdnPublishBuild = build job: 'spark-js-sdk--publish-chat-widget-s3', parameters: [[$class: 'StringParameterValue', name: 'buildNumber', value: currentBuild.number]], propagate: false
                            // if (cdnPublishBuild.result != 'SUCCESS') {
                            //    warn('failed to publish to CDN')
                            //}
                        }
                    }                    
                    cleanup()
                }

                catch (error) {
                  // Sometimes an exception can get thrown without changing the build result
                  // from success. If we reach this point and the result is not UNSTABLE, then
                  // we need to make sure it's FAILURE
                    if (currentBuild.result != 'UNSTABLE') {
                        currentBuild.result = 'FAILURE'
                    }
                    cleanup()
                    throw error
                }
            }
        }
    }
}
