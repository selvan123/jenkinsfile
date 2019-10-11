node 
{
	 stage('GitSCM')
    {
        git url: 'https://github.com/selvan123/petclinic.git'
    }
	
     stage('Initialize')
    {
    
		jdk = tool name: 'jdk'
		env.JAVA_HOME = "${jdk}"
		echo "jdk installation path is: ${jdk}"
		// next 2 are equivalents
		sh "${jdk}/bin/java -version"
		// note that simple quote strings are not evaluated by Groovy
		// substitution is done by shell script using environment
		sh '$JAVA_HOME/bin/java -version'
		def mvnHome = tool 'mvn'
	//	sh "${mvnHome}/bin/mvn -B verify"
    }
   
    
    
    stage('Build Stage')
    {
		def mvnHome = tool 'mvn'
		sh "${mvnHome}/bin/mvn -B verify"
    }
    
     stage('Sonar Analysis') 
	{
		withSonarQubeEnv('sonar') 
		{
		    def mvn = tool 'mvn'
			sh 'mvn clean package sonar:sonar'
			sh 'sleep 10'
		} // SonarQube taskId is automatically attached to the pipeline context
	}
	
	stage("Quality Gate")	
	{
		timeout(time: 5, unit: 'MINUTES')
		{ // Just in case something goes wrong, pipeline will be killed after a timeout
			def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
			if (qg.status != 'OK') 
			{
				error "Pipeline aborted due to quality gate failure: ${qg.status}"
			}
		}
	}
	
	stage('Publish to Nexus')
    {
        sh 'mvn deploy:deploy-file -DgroupId=com.cicd -DartifactId=petclinic -Dversion=${BUILD_NUMBER} -DgeneratePom=true -Dpackaging=war -DrepositoryId=nexus-snapshots -Durl=http://10.0.2.18:8081/nexus/content/repositories/petclinic/  -Dfile=/root/.jenkins/jobs/demo-scripted-pipeline/workspace/target/petclinic.war'
    }
    
     stage("Deploy to Rundeck")
	{
	   step([$class: "RundeckNotifier", 
            includeRundeckLogs: true, 
            jobId: "38f04e85-9da9-49ab-a17a-3a5792cdd666", 
            rundeckInstance: "Rundeck", 
            shouldFailTheBuild: true, 
            shouldWaitForRundeckJob: true, 
            tailLog: true]) 

	}
	

		stage('Selenium Testing')
    {
        git url: 'https://github.com/selvan123/petclinic.git'
        sh "cd /root/.jenkins/jobs/demo-scripted-pipeline/workspace/TestCases"
        wrap([$class: 'Xvfb', displayNameOffset: 0, installationName: 'Xvfb', screen: '1600x1200x32']) 
        {
            sh "mvn test"
        }
        
    }
    
    stage('Performance testing')
    {
        sh "sudo bash"
		sh "rm -rf /root/.jenkins/jobs/demo-scripted-pipeline/workspace/Petclinic.jmx"
		sh "rm -rf /root/.jenkins/jobs/demo-scripted-pipeline/workspace/Petclinic.jtl"
		sh "rm -rf /root/.jenkins/jobs/demo-scripted-pipeline/workspace/Petclinic.xml"
		sh "wget https://s3-us-west-2.amazonaws.com/demo-pipeline-devops/Petclinic.jmx"

		sh "sudo /home/ec2-user/jmeter_new/apache-jmeter-4.0/bin/jmeter.sh -Jjmeter.save.saveservice.output_format=xml -n -t /root/.jenkins/jobs/demo-scripted-pipeline/workspace/Petclinic.jmx -l Petclinic.jtl"
        
    }
    
    stage('Security Testing using ZAP')
    {
        sh "cd /var/lib/docker"
        sh "sudo docker run -t owasp/zap2docker-weekly zap-baseline.py -t http://petclinic.42w36m7wvr.us-west-2.elasticbeanstalk.com/ || true"
        
    }
}
