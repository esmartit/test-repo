    withCredentials([usernamePassword(
            credentialsId: 'github',
            usernameVariable: 'username', passwordVariable: 'gitToken')]) {


        def gitUrl = "github.com/esmartit/test-repo.git"
        def label = "worker-${UUID.randomUUID().toString()}"

        podTemplate(label: label, serviceAccount: 'jenkins',
                containers: [
                        containerTemplate(name: 'gradle', image: 'gradle:6.2.2-jdk8', ttyEnabled: true, command: 'cat'),
                        containerTemplate(name: 'semantic-release', image: 'esmartit/semantic-release:1.0.3', ttyEnabled: true, command: 'cat',
                                envVars: [
                                        envVar(key: 'GITHUB_TOKEN', value: gitToken),
                                        envVar(key: 'DOCKER_HOST', value: 'tcp://dind.devops:2375')])
                ],
                volumes: [
                        hostPathVolume(mountPath: '/home/gradle/.gradle', hostPath: '/tmp/jenkins/.gradle'),
                        emptyDirVolume(mountPath: '/var/lib/docker', memory: false)
                ]
        ) {

            node(label) {
                container('semantic-release') {

                    stage('Prepare release') {

                        container('semantic-release') {
                            stage('Checkout code') {
                                
                                sh "git config --global user.email 'tech@esmartit.es'"
                                sh "git config --global user.name 'esmartit'"
                                
                                checkout([$class: 'GitSCM', 
                                        branches: [[name: '*/${env.CURRENT_BRANCH}'],
                                                    [name: '*/gh-pages']], 
                                        extensions: [],
                                        userRemoteConfigs: [[credentialsId: 'esmartit-github-ssh', url: 'git@github.com:esmartit/test-repo.git']]])
                            }
                            stage('Switch to gh-pages') {
                                echo 'hello there!'
                                
                                sh "git checkout gh-pages"
                                sh "echo test: $label >> index.yaml"
                                sh "git add ."
                                sh "git status"
                                sh "git commit -m \"adding new artifact version: \""                                
                                
                                withEnv(['GIT_SSH_COMMAND=ssh -o StrictHostKeyChecking=no']) {
                                    sshagent(credentials: ['esmartit-github-ssh']) {
                                        sh "git push"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
