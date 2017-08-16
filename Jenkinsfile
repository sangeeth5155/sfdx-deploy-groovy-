import groovy.json.JsonSlurperClassic
node {
    def SFDC_USERNAME
	def DEBUG_ISU=env.SFDX_DEBUG
    def HUB_ORG=env.HUB_ORG_DHC
    def SFDC_HOST=env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID=env.JWT_CRED_ID_DH
	def rmgsplit
	

    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def toolbelt = tool 'toolbelt'
            println SFDC_HOST
            println JWT_KEY_CRED_ID
            println CONNECTED_APP_CONSUMER_KEY
            println HUB_ORG
			println toolbelt


    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
	
        stage('Authorize Scratch Org') {

           echo "started"
		   //echo debug_isu;
            rc = bat returnStatus: true,script: "\"${toolbelt}/sfdx\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername"
            if (rc != 0) { error 'hub org authorization failed' }
			
			rmsg = bat returnStdout: true, script: "\"${toolbelt}/sfdx\" force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
         echo "results in rmg in values--------------------------->"+rmsg
		echo rmsg.getClass().getName()
		println rmsg.length()
		
		/*---------------------------------------
		def sf_username=rmsg.substring(335,369)
		println sf_username
		SFDC_USERNAME=sf_username
		println SFDC_USERNAME
		password_user= bat returnStout: true script: "\"${toolbelt}/sfdx\" force:user:password:generate --targetusername SFDC_USERNAME"
		println password_user
		------------------------------------------------*/
			def rmsg1=rmsg.substring(rmsg.indexOf("{"))
			 
			def robj =new JsonSlurperClassic().parseText(rmsg1)
			print robj
			echo "status checking"			
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
			echo "assign values";
            SFDC_USERNAME=robj.result.username
			println SFDC_USERNAME
            robj = null
		
        }
		
	stage('Push To Test Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}/sfdx\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }
            
        }
		  stage('Run Apex Test') {
				
				
				bat "if not exist ${RUN_ARTIFACT_DIR} md ${RUN_ARTIFACT_DIR}"          
                rc = bat returnStatus: true,script: "\"${toolbelt}/sfdx\" force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'apex test run failed'
                }
				
            }
        
		
        stage('collect results') {
            junit keepLongStdio: true, testResults: 'test/*-junit.xml'
        }
        stage ('Covert to MDAPI')
		{
		bat "if not exist ${MDAPI_FORMAT} md ${MDAPI_FORMAT}"
		rc = bat returnStatus: true,script: "\"${toolbelt}/sfdx\" force:source:convert -d ${MDAPI_FORMAT}"
		bat "git add ${MDAPI_FORMAT}"
		bat "git commit -m 'changes' "
		bat "git push origin HEAD:master"		
		}
    }
	
}
