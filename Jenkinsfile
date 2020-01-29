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
                    // bat 'dotnet "C:\\Program Files (x86)\\Jenkins\\tools\\hudson.plugins.sonar.MsBuildSQRunnerInstallation\\sonar_scanner_dotnet\\SonarScanner.MSBuild.dll" begin  /k:com.nagp2019.nipundavid.3146006.pipeline /n:com.nagp2019.nipundavid.3146006.pipeline /v:1.0'
					// bat 'dotnet "%scannerHome%\\sonar_scanner_dotnet\\SonarScanner.MSBuild.dll" begin  /k:com.nagp2019.nipundavid.3146006.pipeline /n:com.nagp2019.nipundavid.3146006.pipeline /v:1.0'
					sh "dotnet ${scannerHome}/SonarScanner.MSBuild.dll begin /k:com.nagp2019.nipundavid.3146006.pipeline /n:com.nagp2019.nipundavid.3146006.pipeline /v:1.0 "
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
				bat returnStdout: true, script: '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" build --no-cache -t nipundavidsingh07/dotnetcoreapp_nipundavid:101 .'
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
				bat returnStdout: true, script: '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" push nipundavid/dotnetcoreapp_nipundavid:101'
			}
		}
		stage ('Stop Running container')
		{
			steps
			{
				bat '''
					FOR /f "tokens=*" %%i IN ('docker ps -aq') DO (
						IF NOT "%%i"=="0" (
							docker stop %%i
							docker rm %%i
						)
					)
				'''
			}
		}
		stage ('Docker deployment')
		{
			steps
			{
				bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker" run --name dotnetcoreapp_nipundavid -d -p 5017:80 nipundavid/dotnetcoreapp_nipundavid:101'
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
