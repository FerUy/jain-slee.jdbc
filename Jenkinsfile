pipeline {
	agent any

	tools {
	    jdk 'JDK 11'
		maven 'Maven_3.8.5'
	}

	options {
    	buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '10'))
  	}


	parameters {
		string(name: 'RESTCOMM_SLEE_JDBC_VERSION', defaultValue: '7.3.0', description: 'The major version for Restcomm JAIN-SLEE JDBC ')
	}

	environment {
        // ENV_NAME = "${env.BRANCH_NAME}"
        JDBC_BUILD_VERSION ="${params.RESTCOMM_SLEE_JDBC_VERSION}-${BUILD_NUMBER}"
    }

	stages{
		stage("Build") {
			steps {
				echo "Building application..."
				script {
				    // ENV_NAME = '<new name>'
                    JDBC_BUILD_VERSION ="${params.RESTCOMM_SLEE_JDBC_VERSION}-${BUILD_NUMBER}"
                     currentBuild.displayName = "#${JDBC_BUILD_VERSION}"
                     currentBuild.description = "Restcomm SLEE JDBC(${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
				echo "Maven build completed."
			}
		}

		stage('Set Version') {
			steps {
				sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${JDBC_BUILD_VERSION}"
                sh "mvn versions:commit"
			}
		}

		stage("Release") {
			steps {
				echo "Building a release version of #${JDBC_BUILD_VERSION}"
                sh "mvn clean install -Prelease -Drelease.dir=../../../${JDBC_BUILD_VERSION} -Dmaven.test.skip=true"
				echo "Building a release version completed."
			}
		}
		stage('Zip Resources') {
			steps{
				dir("${JDBC_BUILD_VERSION}/resources") {
					sh "zip -r jdbc.zip jdbc"
					echo 'Deleting sub folders'
					sh 'rm -rf jdbc'
				}
			}
		}
		stage('Save Artifacts') {
            steps {
                echo "Archiving Restcomm JDBC version ${JDBC_BUILD_VERSION}"
                archiveArtifacts artifacts: "${JDBC_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
		}

        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
			steps {
                /*sshagent (credentials: ['4e708f2a-8b37-414f-a6d4-787690b87738']) {
                	sh 'ssh -o StrictHostKeyChecking=no -l fer 127.0.0.1 uname -a'
                	sh "scp -r ${JDBC_BUILD_VERSION}/ fer@127.0.0.1:/var/www/html/NAIKERI/jain-slee.jdbc/${JDBC_BUILD_VERSION}/"
                }*/
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.jdbc/${JDBC_BUILD_VERSION}/"
                sh "cp ${JDBC_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.jdbc/${JDBC_BUILD_VERSION}/"
				sh "rm -rf ${JDBC_BUILD_VERSION}"
			}
		}
  }
	post {
		success {
			echo "JAIN-SLEE JDBC successfully built"
		}
		failure {
			echo "Building JAIN-SLEE JDBC failed"
		}
	}
}
