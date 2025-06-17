pipeline {
 	agent any

	options { lock('staging-cluster')}

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
		
		stage('Startup') {
			steps {echo "Build ${env.BUILD_NUMBER} grabbed the lock"}
		}
		stage('Detect Changes') {
			steps{
				scripts{
					def target = env.CHANGE_TARGET ?: 'main'
					def diff = sh(returnStdout: true,
						      script: "git diff --name-only origin/${target}...HEAD")
						    .trim()
						    .split('\n')

					env.NEED_BACKEND = diff.any { it.startsWtih('backend/') }.toString()
					env.NEED_WEB	 = diff.any { it.startsWith('web/') }.toString()
					env.NEED_E2E	 = diff.any { it =~ /\\.(js|css|html$/ }.toString()

					echo """
     					Changed files: ${diff.join(', ')}
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
				stage('Fast') { steps { sh 'echo fast tests && sleep 5' } }
				stage('Slow') { steps { sh 'echo slow tests && sleep 20' } }

			}
		}
	}

		

}
