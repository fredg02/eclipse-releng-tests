/*******************************************************************************
 * Copyright (c) 2019, 2020 Ericsson.
 *
 * This program and the accompanying materials
 * are made available under the terms of the Eclipse Public License 2.0
 * which accompanies this distribution, and is available at
 * https://www.eclipse.org/legal/epl-2.0/
 *
 * SPDX-License-Identifier: EPL-2.0
 *******************************************************************************/
pipeline {
    agent {
        kubernetes {
            label 'tracecompass-build'
            yamlFile 'pod-templates/tracecompass-pod.yaml'
            defaultContainer 'tracecompass'
        }
    }
    options {
        timestamps()
        timeout(time: 4, unit: 'HOURS')
        disableConcurrentBuilds()
    }
    tools {
        maven 'apache-maven-latest'
        jdk 'openjdk-jdk11-latest'
    }
    environment {
        MAVEN_OPTS="-Xms768m -Xmx4096m -XX:+UseSerialGC"
        MAVEN_WORKSPACE_SCRIPTS="../scripts"
        WORKSPACE_SCRIPTS="${WORKSPACE}/.scripts/"
        SITE_PATH="releng/org.eclipse.tracecompass.releng-site/target/repository/"
        RCP_PATH="rcp/org.eclipse.tracecompass.rcp.product/target/products/"
        RCP_SITE_PATH="rcp/org.eclipse.tracecompass.rcp.product/target/repository/"
        RCP_PATTERN="trace-compass-*"
    }
    stages {
        stage('Checkout') {
            steps {
                container('tracecompass') {
                    sh 'mkdir -p ${MAVEN_WORKSPACE_SCRIPTS}'
                    sh 'cp scripts/deploy-rcp.sh ${MAVEN_WORKSPACE_SCRIPTS}'
                    sh 'cp scripts/deploy-update-site.sh ${MAVEN_WORKSPACE_SCRIPTS}'
                    sh 'cp scripts/deploy-doc.sh ${MAVEN_WORKSPACE_SCRIPTS}'
                    checkout([$class: 'GitSCM', branches: [[name: '$GERRIT_BRANCH_NAME']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'BuildChooserSetting', buildChooser: [$class: 'GerritTriggerBuildChooser']]], submoduleCfg: [], userRemoteConfigs: [[refspec: '$GERRIT_REFSPEC', url: '$GERRIT_REPOSITORY_URL']]])
                    sh 'mkdir -p ${WORKSPACE_SCRIPTS}'
                    sh 'cp ${MAVEN_WORKSPACE_SCRIPTS}/deploy-rcp.sh ${WORKSPACE_SCRIPTS}'
                    sh 'cp ${MAVEN_WORKSPACE_SCRIPTS}/deploy-update-site.sh ${WORKSPACE_SCRIPTS}'
                    sh 'cp ${MAVEN_WORKSPACE_SCRIPTS}/deploy-doc.sh ${WORKSPACE_SCRIPTS}'
                }
            }
        }
        stage('Deploy Site') {
            when {
                expression { return params.DEPLOY_SITE }
            }
            steps {
                container('jnlp') {
                    sshagent (['projects-storage.eclipse.org-bot-ssh']) {
                        sh '${WORKSPACE_SCRIPTS}/deploy-update-site.sh ${SITE_PATH} ${SITE_DESTINATION}'
                    }
                }
            }
        }
        stage('Deploy RCP') {
            when {
                expression { return params.DEPLOY_RCP }
            }
            steps {
                container('jnlp') {
                    sshagent (['projects-storage.eclipse.org-bot-ssh']) {
                        sh '${WORKSPACE_SCRIPTS}/deploy-rcp.sh ${RCP_PATH} ${RCP_DESTINATION} ${RCP_SITE_PATH} ${RCP_SITE_DESTINATION} ${RCP_PATTERN} false'
                    }
                }
            }
        }
        stage('Deploy Doc') {
            when {
                expression { return params.DEPLOY_DOC }
            }
            steps {
                container('jnlp') {
                    sshagent (['projects-storage.eclipse.org-bot-ssh']) {
                       sh '${WORKSPACE_SCRIPTS}/deploy-doc.sh'
                    }
                }
            }
        }
    }
}
