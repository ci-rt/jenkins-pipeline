#!/usr/bin/groovy
/* -*- mode: groovy; -*-
 * CI-RT library kernel build test
 */

@Library('CI-RT@master') _

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
		string(defaultValue: '', description: '', name: 'GUI_COMMIT')
	}

	stages {
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
			steps {
				echo "feed database"
				echo "send email"
			}
		}
	}
}
