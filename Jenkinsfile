#!/usr/bin/env groovy
@Library('github.com/stakater/fabric8-pipeline-library@ehancement-configure-release')

def localItestPattern = ""
try {
    localItestPattern = ITEST_PATTERN
} catch (Throwable e) {
    localItestPattern = "com.stakater.kubernetes.assertions.*Tests"
}

def localFailIfNoTests = ""
try {
    localFailIfNoTests = ITEST_FAIL_IF_NO_TEST
} catch (Throwable e) {
    localFailIfNoTests = "false"
}

def versionPrefix = ""
try {
    versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
    versionPrefix = "1.0"
}

def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"


mavenNode(mavenImage: 'openjdk:8') {
    container(name: 'maven') {
        stage('Checkout') {
            checkout scm
        }

        stage('Canary Release') {
            mavenCanaryRelease2 {
                version = canaryVersion
            }
        }

        stage('Integration Tests') {
            mavenIntegrationTest {
                environment = 'Testing'
                failIfNoTests = lFailIfNoTests
                itestPattern = localItestPattern
            }
        }
    }
}

releaseNode {
  try {

    stage('Checkout') {
        checkout scm
    }

    def pipeline = load 'release.groovy'
    def stagedProject

    stage('Stage') {
      stagedProject = pipeline.stage()
    }

    stage('Promote') {
      pipeline.release(stagedProject)
    }

    stage('Promote YAMLs') {
      pipeline.promoteYamls(stagedProject[1])
    }

  } catch (err) {
    error "${err}"
  }
}