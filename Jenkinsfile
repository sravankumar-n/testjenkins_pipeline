def HOSTNAME1 = ""
pipeline {
    agent { label 'dev' }

    environment {
        CLIENT = "CMIG"
        S3_BUCKET = "s3://cmig-ms-properties"
        MS_DIR = "microservices"
        CONSUL_URL = "https://consul.cmig.insurcloud.ca"
        RMQ_DIR = "/var/rabbitmq"
        FILES_DIR = "client/CMIG/cms/deploy/microservices/DEV/files"
 //       HOSTNAME1 = ""
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['DEV01', 'DEV02'], description: 'Environment')
        booleanParam(name: 'CONFIG_UPDATE', defaultValue: false, description: 'Set to true if changes were made to either rabbitmq.conf or definitions.json')
        extendedChoice(bindings: 'image=rabbitmq', defaultValue: '', description: 'Build', groovyClasspath: '', groovyScript: '''import groovy.json.JsonSlurper; def image = binding.variables.get(\'image\'); def result = "http://10.15.16.101:9443/v2/${image}/tags/list".toURL().text; def jsonSlurper = new JsonSlurper(); def object = jsonSlurper.parseText(result); return object.tags.reverse()''', multiSelectDelimiter: ',', name: 'BUILD', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_SINGLE_SELECT', visibleItemCount: 5)
        booleanParam(name: 'CONFIRM', defaultValue: false, description: 'Confirm that you want to deploy')
    }

    stages {
        stage('Set HOSTNAME1') {
            steps {
                script {
		       if ( "${ENVIRONMENT}" == 'DEV01' ) {
                       HOSTNAME1 = "ms1.${ENVIRONMENT}.${CLIENT}.insurcloud.ca"
                     } else if ( "${ENVIRONMENT}" == 'DEV02' ) {
                   HOSTNAME1 = "ms1-${ENVIRONMENT}.nonprod.${CLIENT}.insurcloud.ca"
                   }
                   echo "${HOSTNAME1}"
                }
            }
        }
 /*     
		stage('Push Microservices Files') {
            steps {
                dir(env.FILES_DIR) {
                    script {
                        sh 'aws s3 cp rmq-stack.yml ${S3_BUCKET}/${ENVIRONMENT}/rmq-stack.yml'
                        sh 'aws s3 cp rabbitmq ${S3_BUCKET}/${ENVIRONMENT}/rabbitmq --recursive'
                    }
                }
            }
        }

        stage('Pull Microservices Files') {
            steps {
                sh "ssh ec2-user@${HOSTNAME1} 'aws s3 cp ${S3_BUCKET}/${ENVIRONMENT}/ . --recursive '"
            }
        }
        
        stage('Check for Confirmation') {
            steps {
                script {
                    if(!params.CONFIRM) {
                        currentBuild.result = 'Aborted'
                        error('Lacking Confirmation')
                    }
                }
            }
        }
*/
        stage('Update Config'){
            environment{
                STACK_NAME = "rabbitmq"
//		HOSTNAME = "${HOSTNAME1}"
            }
            steps {
                script {
                    if(params.CONFIG_UPDATE){
			echo "${HOSTNAME1}"
                        //sh 'ssh ec2-user@${HOSTNAME1}  "docker service rm ${STACK_NAME}_rabbitmq"'
                        //sh 'ssh ec2-user@${HOSTNAME1}  "docker config rm ${STACK_NAME}_definitions ${STACK_NAME}_rabbitmq-config"'
                    }
                }
            }
        }

        stage('Start Each Application'){
            environment { 
                RABBITMQ_ERLANG_COOKIE = credentials('dev-rabbitmq-erlang-cookie')
		    HOSTNAME = "${HOSTNAME1}"
            }
            steps {
                dir(env.CLIENT) {
                    script {
                        echo "${HOSTNAME}"
                       // sh 'ssh ec2-user@${HOSTNAME1} "export BUILD=${BUILD}; export ENV=${ENVIRONMENT}; export RABBITMQ_ERLANG_COOKIE=${RABBITMQ_ERLANG_COOKIE}; docker stack deploy --prune --with-registry-auth --compose-file=rmq-stack.yml rabbitmq"'
                    }
                }
            }
        }
    }
}
