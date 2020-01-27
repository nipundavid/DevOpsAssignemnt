pipeline
{
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
				echo "************ restoring dependancies **********"	
                bat "dotnet restore"
			}
		}
		stage ('Start sonarqube analysis')
		{
			steps
			{
				echo "*********** starting sonar analysis ***********"
                withSonarQubeEnv('Test_Sonar')
                {
                    bat 'dotnet "C:\\Program Files (x86)\\Jenkins\\tools\\hudson.plugins.sonar.MsBuildSQRunnerInstallation\\sonar_scanner_dotnet\\SonarScanner.MSBuild.dll" begin  /k:NAGP-Demo-Pipeline /n:NAGP-Demo-Pipeline /v:1.0'
                }
            }
		}
		stage ('build')
		{
			steps
			{
				echo "************* building the solution **********"
                bat "dotnet build -c Release -o WebApplication4/app/build"
			}	
		}
		stage ('SonarQube Analysis end')
		{	
			steps
			{
				echo "*************** Executing Sonar analysis ***********"
                withSonarQubeEnv('Test_Sonar')
			    {
                    bat 'dotnet "C:\\Program Files (x86)\\Jenkins\\tools\\hudson.plugins.sonar.MsBuildSQRunnerInstallation\\sonar_scanner_dotnet\\SonarScanner.MSBuild.dll" end' 
                }
			}
		}
		stage ('Release Artifacts')
		{
			steps
			{
				echo "************** Publishing app ***************"
                bat "dotnet publish -c Release -o WebApplication4/app/publish"
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
				bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" login -u nipundavid -p Markiting1!'
			}
		}
		stage ('Push to DTR')
		{
			steps
			{
				// bat returnStdout: true, script: '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" push inderpalsingh07/dotnetcoreapp_inderpal:101'
			}
		}
		stage ('Stop Running container')
		{
			steps
			// {
			// 	bat '''
			// 		FOR /f "tokens=*" %%i IN ('"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" ps') 
			// 		DO 
			// 		if [[ %%i = *[!\\ ]* ]]; then
			// 			docker stop %%i
			// 		fi
			// 	'''
			}
		}
		stage ('Docker deployment')
		{
			steps
			{
				// bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" run --name dotnetcoreapp_inderpal -d -p 5017:80 inderpalsingh07/dotnetcoreapp_inderpal:101'
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
