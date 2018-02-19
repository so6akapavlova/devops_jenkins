#!groovy​

node(){

	def dockerServerAddress = "tcp://docker.for.mac.host.internal:1234"
	def containers = ['rabbit', 'processor', 'gateway']
	stage 'hello'
		println "Hello world, I am a Groovy script"

	stage 'preparations'
		cleanWs()
		docker.withTool('docker'){
			withDockerServer([uri: dockerServerAddress]){
				containers.each{ container ->
					try {
						sh 'docker rm -f ${container}'
					}
					catch{
						println "${container} is always stopped"
					}

				}

				sh 'docker rmi sobakapavlova/gateway:v1'
				sh 'docker rmi sobakapavlova/processor:v1'
			}
		}
		dir('./source'){
		}

	stage 'get source codes from git'
		dir('./source') {
			git branch: 'master', credentialsId: '104d7369-6ee9-4490-8891-6ef6f309d330', url: 'git@github.com:so6akapavlova/devops_jenkins.git'
}

	stage 'run tests'
		dir('source') {
            withMaven(maven: 'maven') {
                sh 'mvn test > test.log'
                zip 'test.log'
            }
        }

	stage 'build apps'
		dir('source') {
            withMaven(maven: 'maven') {
                sh 'mvn package -Dmaven.test.skip=true'
            }
            docker.withTool('docker') {
                withDockerServer([uri: dockerServerAddress]) {
                    sh 'docker build -t sobakapavlova/processor:v1 -f source/message-processor/Dockerfile_processor source/message-processor/'
                    sh 'docker build -t sobakapavlova/gateway:v1 -f source/message-gateway/Dockerfile_gateway source/message-gateway/'

                    sh 'docker run -d --name rabbit rabbitmq'
                    sh 'docker run -d --name processor sobakapavlova/processor:v1'
                    sh 'docker run -d --name gateway sobakapavlova/gateway:v1'
                    sh 'docker ps'

                }
            }
        }


}
