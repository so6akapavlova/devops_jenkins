#!groovy​

import groovy.json.JsonSlurper


node(){

	def dockerServerAddress = "tcp://docker.for.mac.host.internal:1234"
	def containers = ['rabbit', 'processor', 'gateway']
	def binURL = "https://requestbin.fullcontact.com"
	def buildReport = "BUILD NUMBER ${BUILD_NUMBER}\n\n"
	stage 'preparations'
		cleanWs()
		docker.withTool('docker'){
			withDockerServer([uri: dockerServerAddress]){
				containers.each{ container ->
					try {
						sh "docker rm -f $container; docker rmi sobakapavlova/$container:v1"
						buildReport += "Container ${container} and its image have been removed\n"
					}
					catch(exc){
						buildReport += "There were no ${container} container or its image \n"
					}

				}
			}
		}

	stage 'get source code from git'
		dir('./source') {
			git branch: 'master', credentialsId: '104d7369-6ee9-4490-8891-6ef6f309d330', url: 'git@github.com:so6akapavlova/devops_jenkins.git'
			buildReport += "\nSource code has beed checked out from git\n\n"
		}

	stage 'run tests'
		dir('source') {
            withMaven(maven: 'maven') {
                sh 'mvn test > test.log'
                zip zipFile: 'test.zip', glob: 'test.log'
                archiveArtifacts artifacts: 'test.zip'
                sh "mvn clean"
                buildReport += "Tests were passed, file with test logs was zipped, you can find it with other artifacts\n\n"
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
        buildReport += "The environment was built and containers were started\n\n"
                }
            }
        }
	stage 'create bin'
	    def response = httpRequest( consoleLogResponseBody: true, 
	                                httpMode: 'POST',
	                                url: "${binURL}/api/v1/bins",
	                                validResponseCodes: '100:299').getContent()
	    
	    def binNum = new JsonSlurper().parseText(response).name.toString()
	    buildReport += "Bin ${binNum} was created on ${binURL}\n\n"

	stage 'test the build'
	sleep 40
		def tests = ['curl http://localhost:8080/message -X POST -d \'{"messageId":1, "timestamp":1234, "protocolVersion":"1.0.0", "messageData":{"mMX":1234, "mPermGen":1234}}\'',
                   'curl http://localhost:8080/message -X POST -d \'{"messageId":2, "timestamp":2234, "protocolVersion":"1.0.1", "messageData":{"mMX":1234, "mPermGen":5678, "mOldGen":22222}}\'',
                   'curl http://localhost:8080/message -X POST -d \'{"messageId":3, "timestamp":3234, "protocolVersion":"2.0.0", "payload":{"mMX":1234, "mPermGen":5678, "mOldGen":22222, "mYoungGen":333333}}\'']


		docker.withTool('docker') {
            withDockerServer([uri: dockerServerAddress]) {
				tests.eachWithIndex{ test, index ->
                    sh "docker exec gateway ${test}"
                    def fromProcessor = sh(script:"docker logs --tail 1 processor", returnStdout: true)
                    index++
                    assert fromProcessor.contains("id=${index}")
                    buildReport += "Test ${index}: ${fromProcessor}\n"
                }
            buildReport += "SUCCESS! Ready to be pushed to the bin"
			}
		}

	stage 'post report to the bin'
	    httpRequest( consoleLogResponseBody: true, 
	                 httpMode: 'POST',
	                 url: "${binURL}/${binNum}",
	                 requestBody: "$buildReport")
	    
	    echo "To see build report follow the link ${binURL}/${binNum}?inspect"
}