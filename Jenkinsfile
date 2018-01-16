#!/usr/bin/groovy
/* -*- mode: groovy; -*-
 * CI-RT scheduler Jenkinsfile
 */

pipeline {
	agent any;

	options {
		timestamps()
	}

	parameters {
		string(defaultValue: "patches", description: '', name: 'STASH_PATCHES')
		string(defaultValue: "prodenv", description: '', name: 'STASH_PRODENV')
		string(defaultValue: "rawenvironment", description: '', name: 'STASH_RAWENV')
		string(defaultValue: "compileconf", description: '', name: 'STASH_COMPILECONF')
		string(defaultValue: "https://github.com/ci-rt/test-description.git", description: '', name: 'TESTDESCRIPTION_REPO')
		string(defaultValue: "master", description: '', name: 'GUI_TESTDESCR_BRANCH')
		string(defaultValue: 'localhost:5432', description: 'Hostname of database', name: 'GUI_DB_HOSTNAME')
		string(defaultValue: 'master', description: 'Version of CI-RT Jenkins lib to use', name: 'GUI_CIRT_LIB_VERSION')
		string(defaultValue: '', description: '', name: 'GUI_COMMIT')
	}

	stages {
		stage('load CI-RT library') {
			steps {
				library "CI-RT@${params.GUI_CIRT_LIB_VERSION}"
			}
		}

		stage('inputcheck') {
			steps {
				CIRTinputcheck(params)
				echo "global variables set."
			}
		}

		stage('checkout-testdescription') {
			steps {
				node('master') {
					dir('rawenv') {
						deleteDir();
						git(branch: "${params.GUI_TESTDESCR_BRANCH}",
						    changelog: false, poll: false,
						    url: "${params.TESTDESCRIPTION_REPO}");
						stash("${params.STASH_RAWENV}");
					}
				}
			}
		}

		stage('build CI-RT environment') {
			steps {
				CIRTbuildenv(params.GUI_COMMIT, params);
			}
		}

		stage('compile') {
			steps {
				compiletest(params);
			}
		}

		stage('collect and inform') {
			environment {
				DB_HOSTNAME = "${params.GUI_DB_HOSTNAME}";
			}
			steps {
				echo "feed database"
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
						  credentialsId: 'POSTGRES_CREDENTIALS',
						  usernameVariable: 'DB_USER',
						  passwordVariable: 'DB_PASS']]) {
					feeddatabase(params)
				}
				echo "send email"
			}
		}
	}
}
