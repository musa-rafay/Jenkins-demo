pipeline {
 	agent any

	options { lock('staging-cluster')}

	parameters {
		booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip all test stages')
	}

	triggers {
		GenericTRigger(
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

		stage('Tests') {
			when { expression { !params.SKIP_TESTS } }
			parallel {
				stage('Fast') { steps { sh 'echo fast tests && sleep 5' } }
				stage('Slow') { steps { sh 'echo slow tests && sleep 20' } }

			}
		}
	}

		

}
