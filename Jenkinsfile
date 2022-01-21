#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        /*stage('Deploye Code') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

			println rc
			
			// need to pull out assigned username
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "sfdx force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}else{
			   rmsg = bat returnStdout: true, script: "sfdx force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}
			  
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
        }*/
	    stage('Auth Org') {
        rc = bat returnStatus: true, script: "sfdx force:auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --setalias HubOrg"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
			println rc
        }

        stage('Run Test Code') {

			// need to pull out assigned username
			   rmsg = command "sfdx force:apex:test:run --targetusername HubOrg --resultformat human --codecoverage --testlevel ${TEST_LEVEL}"
			  
            println('Hello from a Job DSL script! ok')
            println(rmsg)
        }


        stage('Export Objects data'){

            emsg = command "sfdx texei:data:export --dataplan ./data-source/data-plan.json --outputdir ./data-source --targetusername HubOrg"
            println('EXPORT TEST')
            println(emsg)
        }

        stage('Import Objects Data into other org'){
            rc2 = command "sfdx auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY_TWO} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setalias OrgDevRebbe"
                    if (rc2 != 0) {
                        error 'Salesforce dev hub org authorization failed.'
                    }
            println rc2
             emsg = command "sfdx texei:data:import --inputdir ./data-source --targetusername OrgDevRebbe"
            println('IMPORT TEST')

        }
    }
}
