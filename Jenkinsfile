//Demo pipeline by Kapil Kaushik

isPullRequest = "${env.BRANCH_NAME}".startsWith("PR")
ARTIFACTORY_BASE = 'http://localhost:8081/artifactory'
artifactory_build_link = ''
artifactory_repo_link = ''

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

    stages {
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
              				  "target": "libs-snapshot-local/${IMAGE}/"
            				}
         			     ]
        			    }"""
				)
				rtPublishBuildInfo (
    					serverId: "sagoon-art1"
				)
            }
	}
        stage('Ansible Deploy to server') {
	    node {
                def pom = readMavenPom file: "pom.xml"
                def repoPath =  "${pom.groupId}".replace(".", "/") + 
                                "/${pom.artifactId}"

        	def version = pom.version

        	if(!FULL_BUILD) { //takes the last version from repo
            	sh "curl -o metadata.xml -s http://${NEXUS_URL}/repository/ansible-meetup/${repoPath}/maven-metadata.xml"
            	version = sh script: 'xmllint metadata.xml --xpath "string(//latest)"',
                             returnStdout: true
        	}	
        	def artifactUrl = "http://${ARTIFACTORY_BASE}/libs-snapshot-local/${repoPath}/${version}/${pom.artifactId}-${version}.war"

        	withEnv(["ARTIFACT_URL=${artifactUrl}", "APP_NAME=${pom.artifactId}"]) {
            	    echo "The URL is ${env.ARTIFACT_URL} and the app name is ${env.APP_NAME}"

            // install galaxy roles
          //  sh "ansible-playbook -i ansible/inventory.ini ansible/deploy.yml -u kadmin"        

            ansiblePlaybook colorized: true, 
            installation: 'ansible',
            inventory: 'ansible/inventory.ini', 
            playbook: 'ansible/deploy.yml' 
//            sudo: true,
//            sudoUser: 'kadmin'
		}	

            }
        }
    }
}
