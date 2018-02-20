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
						sh "docker rm -f $container; docker rmi sobakapavlova/$container:v1"
					}
					catch(exc){
						println "$container is already stopped"
					}

				}
			}
		}
		dir('./source'){
		}

	stage 'get source code from git'
		dir('./source') {
			git branch: 'master', credentialsId: '104d7369-6ee9-4490-8891-6ef6f309d330', url: 'git@github.com:so6akapavlova/devops_jenkins.git'
		}

	stage 'run tests'
		dir('source') {
            withMaven(maven: 'maven') {
                sh 'mvn test > test.log'
                archiveArtifacts artifacts: 'test.log'
            }
        }

	stage 'build'
		dir('source') {
            withMaven(maven: 'maven') {
                sh 'mvn package -Dmaven.test.skip=true'
            }
            docker.withTool('docker') {
                withDockerServer([uri: dockerServerAddress]) {
                    sh 'docker build -t sobakapavlova/processor:v1 -f message-processor/Dockerfile_processor message-processor/'
                    sh 'docker build -t sobakapavlova/gateway:v1 -f message-gateway/Dockerfile_gateway message-gateway/'

                    sh 'docker run -d --name rabbit rabbitmq'
                    sleep 20
                    sh 'docker run -d --link rabbit --name processor sobakapavlova/processor:v1'
                    sh 'docker run -d --link rabbit --name gateway sobakapavlova/gateway:v1'
                    sh 'docker ps'

                }
            }
        }


}
