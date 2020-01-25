pipeline
{
	agent any
	environment
	{
		scannerHome = tool name: 'sonar_scanner_dotnet', type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'   
		JOB_NAME = ''
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
				echo "************ restoring dependancies **********"
				sh "dotnet restore"
				
			}
		}
		stage ('Start sonarqube analysis')
		{
			steps
			{
				withSonarQubeEnv('Test_Sonar')
				{
					sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll begin /k:$JOB_NAME /n:$JOB_NAME /v:1.0 "    
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
				echo "****************** Build Docker image ****************"
			}
		}
		stage ('Push to DTR')
		{
			steps
			{
				echo "***************** Pushing image to Nagarro DTR or Docker Hub **********"
			}
		}
		stage ('Stop Running container')
		{
			steps
			{
				echo "*************** Removing already running conatiners *****************"
			}
		}
		stage ('Docker deployment')
		{
			steps
			{
			   echo "*************** Deploying latest war on Docker Containers **************"
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
