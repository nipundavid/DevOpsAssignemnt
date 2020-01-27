pipeline{
	agent any

environment
{
    scannerHome = tool name: 'sonar_scanner_dotnet', type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'   
}
options
   {
      timeout(time: 1, unit: 'HOURS')
      
      // Discard old builds after 5 days or 5 builds count.
      buildDiscarder(logRotator(daysToKeepStr: '5', numToKeepStr: '5'))
	  
	  //To avoid concurrent builds to avoid multiple checkouts
	  disableConcurrentBuilds()
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
				// sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll begin /k:com.nagp2019.nipundavid.3146006.pipeline /n:com.nagp2019.nipundavid.3146006.pipeline /v:1.0 "    
				bat 'dotnet "C:\\Program Files (x86)\\Jenkins\\tools\\hudson.plugins.sonar.MsBuildSQRunnerInstallation\\sonar_scanner_dotnet\\SonarScanner.MSBuild.dll" begin  /k:com.nagp2019.nipundavid.3146006.pipeline /k:com.nagp2019.nipundavid.3146006.pipeline /v:1.0'
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
}

 post {
        always 
		{
			echo "*********** Executing post tasks like Email notifications *****************"
        }
    }
}
