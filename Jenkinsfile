import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=94efe470-d3e3-4f39-8d81-d75dfa34c3ba',
        'AZURE_TENANT_ID=6718f7d5-d51e-4288-aed4-f785b91acc97']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'jenkins-get-started-wa'
      
      // login Azure
      echo "login azure... 1"

      withCredentials([usernamePassword(credentialsId: 'JenkinsGetStartedSP', passwordVariable: 'CPR8Q~VgiCB57OS-DGvOhUg45WWLxhOBvfs8uat5', usernameVariable: '63905e8f-c130-46c2-8f1d-698f804c9dcc')]) {               
          echo "My client id is $AZURE_CLIENT_ID"
          echo "My client secret is $AZURE_CLIENT_SECRET"
          echo "My tenant id is $AZURE_TENANT_ID"
          echo "My subscription id is $AZURE_SUBSCRIPTION_ID"
      }
      

      
      withCredentials([usernamePassword(credentialsId: 'JenkinsGetStartedSP', passwordVariable: 'CPR8Q~VgiCB57OS-DGvOhUg45WWLxhOBvfs8uat5', usernameVariable: '63905e8f-c130-46c2-8f1d-698f804c9dcc')]) {               
       sh '''                   
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      echo "publish..."
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
