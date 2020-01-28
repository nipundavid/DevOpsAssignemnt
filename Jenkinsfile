pipeline{
	agent any

environment
{
    scannerHome = tool name: 'sonar_scanner_dotnet', type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'   
}
options
   {
      // Append time stamp to the console output.
      timestamps()
      
      timeout(time: 1, unit: 'HOURS')
      
      // Do not automatically checkout the SCM on every stage. We stash what
      // we need to save time.
     // skipDefaultCheckout()
      
      // Discard old builds after 10 days or 30 builds count.
      buildDiscarder(logRotator(daysToKeepStr: '5', numToKeepStr: '5'))
   }
     
stages
{
	stage ('checkout')
    {
		steps
		{
			echo  " ********** Clone starts ******************"
		    checkout scm	 
		}
    }
    stage ('nuget')
    {
		steps
		{
			sh "dotnet restore"	 
		}
    }
	stage ('Start sonarqube analysis')
	{
		steps
		{
			withSonarQubeEnv('Test_Sonar')
			{
				sh "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll begin \\k:com.nagp2019.nipundavid.3146006.pipeline \\n:com.nagp2019.nipundavid.3146006.pipeline \\v:1.0 "
			    
			}
		}
	}
	stage ('build')
	{
		steps
		{
			sh "dotnet build -c Release -o WebApplication4/app/build"
		}	
	}
	stage ('SonarQube Analysis end')
	{	
		steps
		{
		    withSonarQubeEnv('Test_Sonar')
			{
				sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll end"
			}
		}
	}
	stage ('Release Artifacts')
	{
	    steps
	    {
	        sh "dotnet publish -c Release -o WebApplication4/app/publish"
	    }
	}
	stage ('Docker Image')
	{
		steps
		{
		    sh returnStdout: true, script: '/bin/docker build --no-cache -t dtr.nagarro.com:443/dotnetcoreapp_nipundavid:${BUILD_NUMBER} .'
		}
	}
	stage ('Docker Login')
	{
		steps
		{
			sh returnStdout: true, script: '/bin/docker login -u nipundavid -p Markiting1!'
		}
	}
	stage ('Push to DTR')
	{
		steps
		{
			sh returnStdout: true, script: '/bin/docker push dtr.nagarro.com:443/dotnetcoreapp_nipundavid:${BUILD_NUMBER}'
		}
	}
	stage ('Stop Running container')
	{
	    steps
	    {
	        sh '''
                ContainerID=$(docker ps | grep 5000 | cut -d " " -f 1)
                if [  $ContainerID ]
                then
                    docker stop $ContainerID
                    docker rm -f $ContainerID
                fi
            '''
	    }
	}
	stage ('Docker deployment')
	{
	    steps
	    {
	       sh 'docker run --name dotnetcoreapp_nipundavid -d -p 5000:80 dtr.nagarro.com:443/dotnetcoreapp_nipundavid:${BUILD_NUMBER}'
	    }
	}
}

 post {
        always 
		{
			emailext attachmentsPattern: 'report.html', body: '${JELLY_SCRIPT,template="health"}', mimeType: 'text/html', recipientProviders: [[$class: 'RequesterRecipientProvider']], replyTo: 'nipun.david@nagarro.com', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'nipun.david@nagarro.com'
        }
    }
}
