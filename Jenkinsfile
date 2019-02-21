//Demo pipeline by Kapil Kaushik

isPullRequest = "${env.BRANCH_NAME}".startsWith("PR")
ARTIFACTORY_BASE = '192.168.43.97:8081/artifactory'
artifactory_build_link = ''
artifactory_repo_link = ''
MVN_GROUPID = ''
MVN_ARTIFACTID = ''
MVN_VERSION = ''
sonar_link = ''
abortOnFailedQualityGate = true

pipeline {
    agent any
    tools {
        maven 'maven' 
    }
    environment {
	M2_HOME = tool 'maven'
//    	IMAGE = readMavenPom().getArtifactId()
//    	VERSION = readMavenPom().getVersion()
//	artifactUrl = "http://${ARTIFACTORY_BASE}/libs-snapshot-local/${IMAGE}/${VERSION}.war" 
	}
	options {
		disableConcurrentBuilds()
	}
	stages {
/*		stage ('Clean Checkout') {
			steps {
				dir ("${WORKSPACE}") {
					deleteDir()
				}
				checkout scm
			}
		} */
		stage ('Set Variables') {
			steps {
				script {
					def mvnPom = readMavenPom file: 'pom.xml'
					MVN_GROUPID = "${mvnPom.groupId}"
					if (MVN_GROUPID == null || MVN_GROUPID == '' || MVN_GROUPID == 'null') {
						MVN_GROUPID = "${mvnPom.parent.groupId}"
					}
					MVN_ARTIFACTID = "${mvnPom.artifactId}"
					MVN_VERSION = "${mvnPom.version}"
				}
			}
		}
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            }
	stage ('Artifactory') {

			when {

				expression {isPullRequest == false}
			}
			steps {
				rtUpload (
    				serverId: "sagoon-art1",
    				spec:
        			    """{
          			      "files": [
            				{				
              				  "pattern": "target/*.war",
              				  "target": "libs-snapshot-local/${MVN_GROUPID}/${MVN_VERSION}/"
            				}
         			     ]
        			    }"""
				)
				rtBuildInfo (
				    maxBuilds: 2,
				    deleteBuildArtifacts: true
				)
				rtPublishBuildInfo (
    					serverId: "sagoon-art1"
				)
            }
	}
        stage('Ansible Deploy to server') {
        	steps {
                     script {
        		artifactUrl = "http://${ARTIFACTORY_BASE}/libs-snapshot-local/${MVN_GROUPID}/1.0-SNAPSHOT-${env.BUILD_NUMBER}/${MVN_ARTIFACTID}-1.0-SNAPSHOT-${env.BUILD_NUMBER}.war"

        		withEnv(["ARTIFACT_URL=${artifactUrl}", "APP_NAME=${MVN_ARTIFACTID}", "WAR_FILE=${MVN_ARTIFACTID}-1.0-SNAPSHOT-${env.BUILD_NUMBER}.war"]) {
            	    	echo "The URL is ${env.ARTIFACT_URL} and the app name is ${env.APP_NAME}"

            // install galaxy roles
          //  sh "ansible-playbook -i ansible/inventory.ini ansible/deploy.yml -u kadmin"        

            ansiblePlaybook colorized: true, 
            	installation: 'ansible',
            	inventory: 'ansible/inventory.ini', 
            	playbook: 'ansible/deploy.yml' 
//	    	extraVars   : [
//                        ARTIFACT_URL: "${artifactUrl}",
//                            ]
//            sudo: true,
//            sudoUser: 'padmin'
		}	
	      }
            }
        }
    }
}
