
isPullRequest = "${env.BRANCH_NAME}".startsWith("PR")
MVN_GROUPID = ''
MVN_VERSION = ''
MVN_ARTIFACTID = ''
tokens = "${env.JOB_NAME}".tokenize('/')
repoOrg = tokens[0]
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
              				  "target": "libs-snapshot-local/com/sagoon/app/"
            				}
         			     ]
        			    }"""
				)
/*				rtMavenResolver (
			    		id: 'resolver-unique-id',
    					serverId: 'sagoon-art1',
    					releaseRepo: 'libs-release',
    					snapshotRepo: 'libs-snapshot'
				)  
 
				rtMavenDeployer (
    					id: 'deployer-unique-id',
    					serverId: 'sagoon-art1',
    					releaseRepo: 'libs-release-local',
    					snapshotRepo: "libs-snapshot-local"
				)
				rtMavenRun (
				// Tool name from Jenkins configuration.
    					tool: "${M2_HOME}/bin/mvn",
    					pom: 'pom.xml',
    					goals: 'clean install',
    					// Maven options.
    					//opts: '-Xms1024m -Xmx4096m',
    					resolverId: 'resolver-unique-id',
    					deployerId: 'deployer-unique-id',
				) */	
				rtPublishBuildInfo (
    					serverId: "sagoon-art1"
				)

            }
	}
    }
}
