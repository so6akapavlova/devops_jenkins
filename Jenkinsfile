#!groovy​

import groovy.json.JsonSlurper


node(){

	def dockerServerAddress = "tcp://docker.for.mac.host.internal:1234"
	def containers = ['rabbit', 'processor', 'gateway']

	stage 'preparations'
		cleanWs()
		docker.withTool('docker'){
			withDockerServer([uri: dockerServerAddress]){
				containers.each{ container ->
					try {
						sh "docker rm -f $container; docker rmi sobakapavlova/$container:v1"
					}
					catch(exc){
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
                zip zipFile: 'test.zip', glob: 'test.log'
                archiveArtifacts artifacts: 'test.zip'
                sh "mvn clean"
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
	stage 'create bin'
	    def response = httpRequest( consoleLogResponseBody: true, 
	                                httpMode: 'POST',
	                                url: "https://requestbin.fullcontact.com/api/v1/bins",
	                                validResponseCodes: '100:299').getContent()
	    
	    def binNum = new JsonSlurper().parseText(response).name.toString()
	stage 'test the build'
		def tests = ['curl http://localhost:8080/message -X POST -d \'{"messageId":1, "timestamp":1234, "protocolVersion":"1.0.0", "messageData":{"mMX":1234, "mPermGen":1234}}\'',
					'curl http://localhost:8080/message -X POST -d \'{"messageId":2, "timestamp":2234, "protocolVersion":"1.0.1", "messageData":{"mMX":1234, "mPermGen":5678, "mOldGen":22222}}\''
					'curl http://localhost:8080/message -X POST -d \'{"messageId":3, "timestamp":3234, "protocolVersion":"2.0.0", "payload":{"mMX":1234, "mPermGen":5678, "mOldGen":22222, "mYoungGen":333333}\''
					]
		tests.each{ test ->
			docker.withTool('docker') {
                withDockerServer([uri: dockerServerAddress]) {
                    sh "docker exec gateway ${test}"
                    def fromProcessor = sh "docker logs --tail 1 processor"
                    println fromProcessor
                }
		}

}
