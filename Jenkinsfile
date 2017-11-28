@Library('pipeline')
// import com.ibm.omaas.CfApp

// artifactory = Artifactory.server('omaas-artifactory')
// art(artifactory, 'omaas-staging')
// cloudTools('cloudtools-git-id','./servicemanifest.json')

node('cem_slave') {
    def version         = "1.0.4"
    def buildClassifier = env.BUILD_TIMESTAMP.replaceAll(/[ :-]/,'')
    def pipelineName    = env.JOB_NAME?.split('/')[0]
    def urlLinkBuildSummary = "${env.JENKINS_URL}job/${pipelineName}/job/${env.BRANCH_NAME}/${env.BUILD_NUMBER}/"
    def gitCommit = sh(returnStdout: true, script: "git rev-parse HEAD").trim()

    def registerInternalDeploys = env.REGISTER_INTERNAL_DEPLOY?.toLowerCase()?.equals("true")
    echo "Register deploys to internal bluemix systems is set to ${registerInternalDeploys}"

    def targetEnv   = "edge"
    def bluemixApi  = "https://api.stage1.ng.bluemix.net"
    def domain      = "stage1.mybluemix.net"
    def org         = "itsm_dev"
    def space       = "emaas-test-staging"
    def host        = "edge-eventpreprocessor"
    def doDeploy    = false
    def cfApp
    def deployConfig

    /*
     * Set project options do delete old builds locally and artifacts in artifactory.
     * Release versions should be moved to the release section of artifactory so
     * will not be deleted.
     */
    // properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '30', artifactNumToKeepStr: '30',
    //         daysToKeepStr: '15', numToKeepStr: '15')),
    //             [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
    //             pipelineTriggers([])])

    // manager.buildSuccess();

    withEnv([
            // You need to specify 3 required environment variables and your bluemix credentials first, they are going to be used for the following IBM Cloud DevOps steps
            'IBM_CLOUD_DEVOPS_ORG=xunrongli@us.ibm.com',
            'IBM_CLOUD_DEVOPS_APP_NAME=eventpreprocessor',
            'IBM_CLOUD_DEVOPS_TOOLCHAIN_ID=1'
    ]) {
        //specify your bluemix credentials, please use "IBM_CLOUD_DEVOPS_CREDS_USR" for usernameVariable, "IBM_CLOUD_DEVOPS_CREDS_PSW" for passwordVariable
        withCredentials([string(credentialsId: 'DOIIntegration', variable: 'IBM_CLOUD_DEVOPS_API_KEY')]) {
            stage ('Check out') {
                checkout scm
                // deployConfig = load 'buildsrc/deploy.groovy'

                // cfApp = deployConfig.getApp(targetEnv)
                if (env.BRANCH_NAME == "develop") {
                    doDeploy = true

                    // cfApp = deployConfig.getApp(targetEnv)
                } else if (env.BRANCH_NAME.startsWith("release-")) {
                    doDeploy = true
                    // space = "cem-" + env.BRANCH_NAME + "-staging"
                    targetEnv = "production"
                    host = "cem-" + deployConfig.serviceName + "-" + env.BRANCH_NAME.replaceAll("\\.", "-") + "-staging"

                    // cfApp = new CfApp(
                    //         deployConfig.serviceName,
                    //         deployConfig.artifactId,
                    //         deployConfig.services,
                    //         [
                    //                 targetEnv: targetEnv,
                    //                 memory: "512M",
                    //                 instances: 1,
                    //                 appDir: "src",
                    //                 host: host,
                    //                 appVars: ["CNAME=internal","ENABLE_SWAGGER_UI=1"]
                    //         ])
                }
            }

            stage ('Build') {
                withEnv(["GIT_COMMIT=${gitCommit}",
                         'GIT_BRANCH=env.BRANCH_NAME',
                         "GIT_REPO=https://github.ibm.com/OMaaS/eventpreprocessor"]) {
                    try {
                        // dir ('src') {
                        //     npmInstall.forMicroService("${cfApp.artifactId}-${version}-${buildClassifier}")
                        // }

                        // use "publishBuildRecord" method to publish build record
                        publishBuildRecord gitBranch: "${GIT_BRANCH}", gitCommit: "${GIT_COMMIT}", gitRepo: "${GIT_REPO}", result:"SUCCESS"
                    }
                    catch (Exception e) {
                        publishBuildRecord gitBranch: "${GIT_BRANCH}", gitCommit: "${GIT_COMMIT}", gitRepo: "${GIT_REPO}", result:"FAIL"
                    }
                }
            }

            stage('Test') {
                // Get credentials required for local test, will be deleted when test is complete
                checkout([$class: 'GitSCM',
                          userRemoteConfigs: [
                                  [url: 'https://github.ibm.com/OMaaS/external-config.git',
                                   refspec: 'refs/heads/master:refs/remotes/origin/master',
                                   credentialsId: 'noiea_scan_token']],
                          extensions: [[$class: 'RelativeTargetDirectory',
                                        relativeTargetDir: 'external-config']]])

                try {
                    // sh """#!/bin/bash
                    //     . ~/.nvm/nvm.sh
                    //     nvm use v6
                    //     xvfb-run -e xvfb-err.log --server-args="-screen 0, 1024x768x24 -ac +extension RANDR" bash ./test.sh -x
                    // """
                } catch (Exception err ) {
                    //Testing failed. We want to keep running because we want to
                    //Capture the build results and logs.
                    echo 'Failed to run tests'
                    manager.buildFailure();
                }

                dir('external-config') {
                    deleteDir()
                }
            }

            if (!doDeploy) {
                def override = sh returnStdout: true, script: "git --no-pager log -n 1 --pretty='%s' | tail -2 | head -1"
                echo 'deploy override is ' + override
                // if (override.trim() == "deploy") {
                //     doDeploy = true
                //     cfApp.serviceName = "${cfApp.serviceName}-" + getCommitterEmailName().replaceAll(/[.]/,'').toLowerCase()
                //     cfApp.host = "${targetEnv}-${cfApp.serviceName}"
                // }
            }

            if (doDeploy) {
                /*
                 * If we are a build destined for production then register with bluemix service manifest and embed a runtime
                 * that will tell them when we start.
                 */
                // if ( env.BRANCH_NAME.startsWith("release-") || registerInternalDeploys) {
                //     cloudTools.buildAndRegister(cfApp.appDir)
                // }
                // lock (resource: "${serviceName}-deploy") {
                //     stage ('Deploy') {
                //         rc = deployService.cfApp(cfApp, bluemixApi, domain, org, space)
                //         if (rc==0) {
                //             manager.addInfoBadge('Deployed to ' + targetEnv)
                //         } else {
                //             manager.addErrorBadge("Deploy to ${targetEnv} failed");
                //             error("Deploy of ${serviceName} to ${targetEnv} failed")
                //         }
                //     }
                //     stage ('Basic connectivity test') {
                //         def testURL = "https://${host}.${domain}/docs/events/v1/"
                //         sh """#!/bin/bash
                //             buildsrc/pingMicroService.sh ${testURL} ${cfApp.serviceName} > src/test-results/basicConnectivity.xml || echo Tests failed
                //         """
                //     }
                // }
            }

            //We need to wait until both the ping tests and unit tests have run to archive, because Jenkins doesn't work correctly
            //with multiple test archive steps
            try {
                step([$class: 'JUnitResultArchiver', testResults: 'src/test-results/*.xml'])
            } catch (Err) {
                //Failed to find any JUNIT files
                echo "No test results found"
                manager.buildFailure();
            }

            if (getBuildResult()!='SUCCESS') {
                archiveArtifacts artifacts: '**/*.log', excludes: null
            }

            echo "Build results ${getBuildResult()}"

            if ((env.BRANCH_NAME == "develop"|| env.BRANCH_NAME.startsWith("release-")) && getBuildResult()!='FAILURE') {
                stage('Publish') {
                    def baseVersion = ''
                    if (env.BRANCH_NAME.startsWith("release-")) {
                        baseVersion = env.BRANCH_NAME.replace("release-", "")
                    }

                    // sh """
                    //   zip -r target/${cfApp.artifactId}-${version}-${buildClassifier}.zip . -x "node_modules/**" -x "target/**" -x "lib/**" -x ".git/**"
                    // """

                    if (env.BRANCH_NAME == "develop" ) {
                        echo "build retention info is specified"
                        // art.pushBuild(cfApp.artifactId, version, buildClassifier, baseVersion, env.BRANCH_NAME,
                        //         deployConfig.buildRetentionInfo)
                    } else {
                        // art.pushBuild(cfApp.artifactId, version, buildClassifier, baseVersion, env.BRANCH_NAME)
                    }
                    def buildResults = getBuildResult()

                    def release = """{
                        "tag_name": "${cfApp.artifactId}-${version}-${buildClassifier}",
                        "target_commitish": "${env.BRANCH_NAME}",
                        "name": "${cfApp.artifactId} v${version} for OMaaS",
                        "body": "Release of ${cfApp.artifactId} app v${version}.<br />Build status ${buildResults}.<br /><a href='${urlLinkBuildSummary}'>Build Summary</a>",
                        "draft": false,
                        "prerelease": true
                    }"""

                    withCredentials([[$class: 'StringBinding', credentialsId: 'noiea-github-token', variable: 'token']]) {
                        // sh """
                        //     curl -sS -XPOST -H 'Content-Type:application/json' -H 'Accept:application/json' -u noiea:\$token \
                        //     --data '${release}' https://github.ibm.com/api/v3/repos/OMaaS/${cfApp.gitRepositoryName}/releases
                        // """
                    }
                }

                stage ('Image') {
                    sh("sudo chown -R jenkins:jenkins /var/run/docker.sock")

                    def registryUrl = 'https://omaas-staging-docker.artifactory.swg-devops.com'
                    if (env.BRANCH_NAME.startsWith("release-")) {
                        registryUrl = 'https://omaas-release-docker.artifactory.swg-devops.com'
                    }

                    docker.withRegistry(registryUrl, 'cem-artifactory-token') {
                        def image = docker.build("${cfApp.artifactId}:${version}-${buildClassifier}")
                        image.push("${version}-${buildClassifier}")
                        image.push("latest")
                    }

                    sh("docker rmi -f ${cfApp.artifactId}:${version}-${buildClassifier}")
                }
            }
        }
    }
}

def getBuildResult() {
    return currentBuild.result ?: 'NONE'
}

def getCommitterEmailName() {
    lastCommitter = sh returnStdout: true, script: "git --no-pager show -s --pretty='%ae'"
    return lastCommitter.split('@')[0]
}