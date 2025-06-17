pipeline {
 	agent any

	parameters {
		booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip all test stages')
	}

	triggers {
		GenericTrigger(
			token: 'gh',
			printContributedVariables: true,
			printPostContent: true
		)
		
	}
	
	stages {
		
		stage('Hello') {
			steps {echo "Webhook reached branch ${env.BRANCH_NAME}"}
		}

		stage('Checkout') {
			steps {
				checkout scm
			}
		}
		
		stage('Startup') {
			steps {echo "Build ${env.BUILD_NUMBER} grabbed the lock"}
		}
		
		stage('Detect Changes') {
			steps{
				script {
					def target = env.CHANGE_TARGET ?: 'main'

					sh "git fetch --no-tags --depth=50 origin${target}"
					
					def rawDiff = sh(returnStdout: true,
						      script: "git diff --name-only origin/${target}...HEAD"
						     ).trim()

					def diff = rawDiff.split('\\r?\n'
								).collect{ it.replaceFirst(/^\\./,'') 
								}.findAlld { it }

					env.NEED_BACKEND = diff.any { it.startsWith('backend/') }.toString()
					env.NEED_WEB	 = diff.any { it.startsWith('web/') }.toString()
					env.NEED_E2E	 = diff.any { it.startsWith('e2e/')	||
								      it =~ /\\.(js|css|html)$/ }.toString()

					echo """
     					Changed files:\n${diff.join(', ')}
	  				NEED_BACKEND =  ${env.NEED_BACKEND}
       					NEED_WEB     =  ${env.NEED_WEB}
	    				NEED_E2E     =  ${env.NEED_E2E}
	 				"""
				}
			}
		}

		stage('Tests') {
			when { expression { !params.SKIP_TESTS } }
			parallel {
				stage('Backend') {
					when { expression { env.NEED_BACKEND == 'true'} }
					steps {
						sh 'python -m pytest backend'
					}
				}
				stage('Web') {
					when { expression { env.NEED_WEB == 'true'} }
					steps {
						sh 'npm --prefix web install --loglevel=error'
						sh 'npm --prefix web test'
					}
				}
			}
		}
		
		stage('E2E') {
			when { expression { env.NEED_E2E = 'true' && !params.SKIP_TESTS } }
			options { lock('staging-cluster') }
			steps   {
				  sh 'bash ./e2e/run_e2e.sh'
				}
		}
	}
}
