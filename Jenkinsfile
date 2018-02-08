#!groovy​

node(){
	stage 'hello'
		println "Hello world, I am a Groovy script"

	stage 'created new folder'
		dir('./source'){

		}

	stage 'get source codes from git'
		dir('./source') {
			git branch: 'master', credentialsId: '104d7369-6ee9-4490-8891-6ef6f309d330', url: 'git@github.com:so6akapavlova/devops_jenkins.git'
}

	stage 'run tests'
		dir('source') {
            withMaven(maven: 'maven') {
                sh 'mvn package -Dmaven.test.skip=true'
            }
            docker.withTool('docker') {
                withDockerServer([uri: 'tcp://docker.for.mac.host.internal:1234']) {
                    sh 'docker ps'
                }
            }
        }


}
