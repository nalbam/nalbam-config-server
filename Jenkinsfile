#!groovy

//echo "JOB_NAME    ${env.JOB_NAME}"
//echo "BRANCH_NAME ${env.BRANCH_NAME}"

properties([buildDiscarder(logRotator(daysToKeepStr: '60', numToKeepStr: '10')), pipelineTriggers([])])

def toast = 2;

node {
    stage('Checkout') {
        checkout scm
    }

    stage('Build') {
        if (env.BRANCH_NAME == 'master') {
            sh '~/toaster/toast.sh version next'
        }
        if (toast == 1) {
            mvn 'clean deploy -B -e'
        } else {
            mvn 'clean package -B -e'
        }
    }

    stage('Code Analysis') {
        mvn 'checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs -B -e'
        step([$class: 'CheckStylePublisher', pattern: 'target/checkstyle-result.xml'])
        step([$class: 'FindBugsPublisher', pattern: 'target/findbugsXml.xml'])
        step([$class: 'PmdPublisher', pattern: 'target/pmd.xml'])
        step([$class: 'DryPublisher', pattern: 'target/cpd.xml'])
        step([$class: 'TasksPublisher', high: 'FIXME', low: '', normal: 'TODO', pattern: 'src/**/*.java, src/**/*.php'])
    }

    stage('Publish') {
        archive 'target/*.jar, target/*.war'
        sh '~/toaster/toast.sh version save'
        if (toast == 1) {
            sh '/data/deploy/bin/version-dev.sh'
        }
    }
}

// Run Maven from tool "mvn"
void mvn(def args) {
    // Get the maven tool.
    // ** NOTE: This 'M3' maven tool must be configured
    // **       in the global configuration.
    def mvnHome = tool 'M3'

    sh "${mvnHome}/bin/mvn ${args}"
}
