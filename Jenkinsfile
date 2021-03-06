node('master') {
    currentBuild.result="SUCCESS"
     def server = Artifactory.server 'JFrog'
     def mvnHome
    try {
        stage ('Check-Out'){
            slackSend color: '#4171f4', message: 'Deployment started on Pivotal Cloud Foundry '
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '3a22a56c-a9fa-4ec7-885e-88085f535991', url: 'https://smetal1@bitbucket.org/ctsihg/ihg-microservices-demo']]])
        }
        stage ('SonarQube'){
            withSonarQubeEnv('SonarQube'){
                sh 'mvn clean package -DskipTests=true sonar:sonar'
            }
        }

        stage ('Artifactory-Push'){
           
            def uploadSpec = """{
                            "files": [
                             {
                                "pattern": "./hotel-details-service/target/*.jar",
                                "target": "ihg-repo/"
                             },
                             {
                                 "pattern": "./inventory-service/target/*.jar",
                                "target": "ihg-repo/"
                             },
                             {
                                 "pattern": "./price-service/target/*.jar",
                                "target": "ihg-repo/"
                             },
                              {
                                 "pattern": "./property-search-service/target/*.jar",
                                "target": "ihg-repo/"
                             },
                             {
                                 "pattern": "./config-server/target/*.jar",
                                "target": "ihg-repo/"
                             }
                             
                            ]
                            }"""
            server.upload(uploadSpec)

        }
        stage('Artifactory-Pull'){
            def downloadSpec = """{
            "files": [
                         {
                            "pattern": "ihg-repo/config-server-0.0.1-SNAPSHOT.jar",
                            "target": "./config-server/target/"
                        },
                        {
                        
                            "pattern": "ihg-repo/hotel-details-service-0.0.1.jar",
                            "target": "./hotel-details-service/target/"
                        },
                          {
                        
                            "pattern": "ihg-repo/inventory-service-0.0.1.jar",
                            "target": "./inventory-service/target/"
                        },
                        {
                        
                            "pattern": "ihg-repo/price-service-0.0.1.jar",
                            "target": "./price-service/target/"
                        },
                         {
                        
                            "pattern": "ihg-repo/property-search-service-0.0.1.jar",
                            "target": "./property-search-service/target/"
                        }
                    ]
            }"""
            server.download(downloadSpec)
            
           
        }
        stage('PCF_Deployment'){
            sh 'chmod +x cf_deployment.sh'
            sh './cf_deployment.sh'
            slackSend color: '#42f44e', message: "Deployment Successfull on Pivotal Cloud Foundry!! Details: '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})  Visit: <https://ihgdemo.app.dev.digifabricpcf.com/|IHG Dashboard>"
        }
             stage('Functional Testing- API') {
             slackSend color: '#f49242', message: "Post Deployment Testing Started!! Details:'${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) "
       build job: 'IHG_Test_Automation_Job', parameters: [string(name: 'MavenCommand', value: 'mvn clean test -PrunTestsForAPIOnPCF -DprofileIdEnabled=true')]
  }
  
  stage('Early Performance Testing- API'){
        build job: 'Early Performance Testing - API Job' 
    
  }
       
          stage('Smoke Test') {
          build job: 'IHG_Test_Automation_Job', parameters: [string(name: 'MavenCommand', value: 'mvn clean test -PrunTestsForWebOnPCF -DprofileIdEnabled=true -Dcucumber.options="--tags @Smoke')]
  }

  
   stage('Functional Testing(Desktop & Mobile)') {
              build job: 'IHG_Test_Automation_Job', parameters: [string(name: 'MavenCommand', value: 'mvn clean test -PrunTestsForWebOnPCF -DprofileIdEnabled=true -Dcucumber.options="--tags @WebUI"')]
  }
     stage('Early Performance Testing') {
     build job: 'Early Performance Testing - UI Job'
      slackSend color: '#0ccc78', message: "Post Deployment Testing Completed!! Details:'${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) Visit : <https://ihgdemo.app.dev.digifabricpcf.com/|IHG Dashboard>"
  }


       
    } catch(err){
        slackSend color: '#e2062b', message: "Deployment Failed on Pivotal Cloud Foundry!! Details:'${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) "
        currentBuild.result = "FAILURE"
        throw err
    }
}