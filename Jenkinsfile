pipeline{
    //Directives
    agent any
    tools {
        maven 'Maven-Project'
    }
    environment{
       ArtifactId = readMavenPom().getArtifactId()
       Version = readMavenPom().getVersion()
       Name = readMavenPom().getName()
       GroupId = readMavenPom().getGroupId()
    }

    stages {
        // Specify various stage with in stages

        // stage 1. Build
        stage ('Build'){
            steps {
                sh 'mvn clean install package'
            }
        }

        // Stage2 : Testing
        stage ('Test'){
            steps {
                echo ' testing......'

            }
        }

        // Stage3 : Publish the artifacts to Nexus
        stage ('Publish to Nexus'){
            steps {
                script {

                def NexusRepo = Version.endsWith("SNAPSHOT") ? "DevopsLab-SnapShot" : "DevopsLab-Release"

                nexusArtifactUploader artifacts:
                [[artifactId: "${ArtifactId}",
                classifier: '',
                file: "/var/lib/jenkins/workspace/Maven-Project-CI-CD-PipeLine/webapp/target/webapp.war",
                type: 'war']],
                credentialsId: '81582bb0-3533-4c26-a58d-ca36404ff39b',
                groupId: "${GroupId}",
                nexusUrl: '52.6.36.19:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: "${NexusRepo}",
                version: "${Version}"
             }
            }
        }

        // Stage 4 : Print some information
        stage ('Print Environment variables'){
                    steps {
                        echo "Artifact ID is '${ArtifactId}'"
                        echo "Version is '${Version}'"
                        echo "GroupID is '${GroupId}'"
                        echo "Name is '${Name}'"
                    }
                }

        // Stage5 : Deploying Build Artifact to Tomcat
        stage ('Deploy to Tomcat'){
            steps {
                echo "Deploying ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'Ansible_Controller',
                    transfers: [
                        sshTransfer(
                                cleanRemote:false,
                                execCommand: 'ansible-playbook /home/ansibleadmin/playbook/download-war-deploy-tomcat.yaml -i /home/ansibleadmin/playbook/hosts',
                                execTimeout: 120000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: false)
                    ])
            }
        }

        // Stage6 : Deploying Build Artifact to Docker
        stage ('Deploy to Docker'){
            steps {
                echo "Deploying ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'Ansible_Controller',
                    transfers: [
                        sshTransfer(
                                cleanRemote:false,
                                execCommand: 'ansible-playbook /home/ansibleadmin/playbook/download-war-deploy-docker.yaml -i /home/ansibleadmin/playbook/hosts',
                                execTimeout: 180000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: false)
                    ])
            }
        }
  }

}
