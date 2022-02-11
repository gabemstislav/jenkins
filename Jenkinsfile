#!groovy
import groovy.json.JsonSlurperClassic
node {


    def SFDC_USERNAME = env.HUB_ORG_DH
    def TEST_LEVEL='RunLocalTests'
    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def CONNECTED_APP_CONSUMER_KEY_TWO=env.CONNECTED_APP_CONSUMER_KEY_DH_PROD
    def USERNAME2=env.HUB_ORG_DH_PROD


withEnv(["HOME=${env.WORKSPACE}"]) {
        
    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Auth Org') {

        rc = command "sfdx force:auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --setalias HubOrg"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
			println rc
        }

/*
        stage('Run Test Code') {

			// need to pull out assigned username
			   rmsg = command "sfdx force:apex:test:run --targetusername HubOrg --resultformat human --codecoverage --testlevel ${TEST_LEVEL}"
			  
            println('Hello from a Job DSL script! ok')
            println(rmsg)
        }*/


        stage('Export Objects data'){
		emsg = command "sfdx texei:package:dependencies:install"
            emsg = command "sfdx texei:data:export --dataplan ./data-source/data-plan.json --outputdir ./data-source --targetusername HubOrg"
            println('EXPORT TEST')
            println(emsg)
        }

        stage('Import Objects Data into other org'){
            rc2 = command "sfdx auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY_TWO} --username ${USERNAME2} --jwtkeyfile ${jwt_key_file} --setalias OrgDevRebbe"
                    if (rc2 != 0) {
                        error 'Salesforce dev hub org authorization failed.'
                    }
            println rc2
             emsg = command "sfdx texei:data:import --inputdir ./data-source --targetusername OrgDevRebbe"
            println('IMPORT TEST')

        }
    
    }
}
   
}

    def command(script) {
        if (isUnix()) {
            return sh(returnStatus: true, script: script);
        } else {
            return bat(returnStatus: true, script: script);
        }
    } 


