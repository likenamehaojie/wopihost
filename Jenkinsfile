#!groovy

def projectProperties = [

        //只保留5个构建记录
        [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '5']],

        //参数化构建
        parameters([
                choice(name: 'action', choices: 'NoAction\nPublishToMaven\nPublishAndDeploy', description: '操作类型'),
                choice(name: 'env', choices: 'dev\ntest', description: '发布环境')
        ])

]
properties(projectProperties)

//maven仓库配置
env.MAVEN_USERNAME = 'username'
env.MAVEN_PASSWORD = 'password'

//各个环境的host配置
def hosts = ['dev' : '127.0.0.1',
             'test': '127.0.0.2']

//分支黑名单（分支名出现在白名单中将不会被checkout）
def branchBlacklist = []

//分支白名单（分支名或分支名的前缀（以'-'分割）出现在白名单中才会被checkout）
def branchWhitelist = ['master', 'develop', 'release', 'feature', 'hotfix']

//启用黑名单或白名单开关：-1表示启用黑名单；1表示启用白名单；0表示都不启用
def blackOrWhitelist = 1


node {

    echo "======================================================="
    echo "用户选择的操作为：${params.action}"
    echo "用户选择的环境为：${params.env}"
    echo "当前启用：${blackOrWhitelist == 0 ? '不启用黑白名单' : blackOrWhitelist == 1 ? '白名单'+branchWhitelist : '黑名单'+branchBlacklist}"
    permission = canGetBranch(blackOrWhitelist, branchBlacklist, branchWhitelist)
    echo "分支[${env.BRANCH_NAME}]：" + "${permission == true ? '已授权，正在启动后续流程...' : '未授权，正在停止后续流程...'}"
    echo "======================================================="

    if (params.action != "NoAction" && permission) {

        env.ENV = params.env

        stage('Checkout') {
            checkout scm
        }

        stage('Build') {
            sh "./gradlew -Penv=${env.ENV} clean build"
        }

        stage('Publish') {
            env.REPO_URL = sh(returnStdout: true, script: "./gradlew -Penv=${env.ENV} publish | grep -oP -m 1 'http.*?zip'").trim()
            echo "Publish to url: ${env.REPO_URL}"
            def packageStr = env.REPO_URL.split('/')[-1]
            currentBuild.description = "Build Environment: ${env.ENV}; <br/> <a href=${env.REPO_URL}>${packageStr}</a>"
        }

        if (params.action == "PublishAndDeploy") {
            stage('Deploy') {
                sh "ssh root@${hosts.get(params.env)} 'bash -x -s' < ./deploy.sh " + "${env.REPO_URL}"
            }
        }

    }
}

def canGetBranch(blackOrWhitelist, branchBlacklist, branchWhitelist) {
    res = false
    if (blackOrWhitelist == 0) {
        res = true
    } else if (blackOrWhitelist == -1) {
        if (!branchBlacklist.contains(env.BRANCH_NAME)) res = true
    } else if (blackOrWhitelist == 1) {
        if (branchWhitelist.contains(env.BRANCH_NAME)) res = true
        for (branchName in branchWhitelist){
            if(env.BRANCH_NAME.startsWith(branchName)) res = true
        }
    }
    res
}
