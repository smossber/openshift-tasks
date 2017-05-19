#!groovy


def gitRepo  = 'http://gogs-moss-cicd.apps.advdev.openshift.opentlc.com/moss/openshift-tasks.git'
def sonarUrl = 'http://sonarqube-moss-cicd.apps.advdev.openshift.opentlc.com'
def nexusUrl = 'http://nexus3-moss-cicd.apps.advdev.openshift.opentlc.com/repository/releases'

// Run this node on a Maven Slave
// Maven Slaves have JDK and Maven already installed
node('maven') {
  // Define Maven Command. Make sure it points to the correct settings for our
  // Nexus installation. The file nexus_openshift_settings.xml needs to be in the
  // Source Code repository.
  def mvnCmd = "mvn -s ./nexus_settings.xml"

  stage('Checkout Source') {
    // Get Source Code from SCM (Git) as configured in the Jenkins Project
    // Next line for inline script, "checkout scm" for Jenkinsfile from Gogs
    //git 'http://gogs.xyz-gogs.svc.cluster.local:3000/CICDLabs/openshift-tasks.git'
    //checkout scm
	git "${gitRepo}"

  }

  // The following variables need to be defined at the top level and not inside
  // the scope of a stage - otherwise they would not be accessible from other stages.
  // Extract version and other properties from the pom.xml
  def groupId    = getGroupIdFromPom("pom.xml")
  def artifactId = getArtifactIdFromPom("pom.xml")
  def version    = getVersionFromPom("pom.xml")

  // Using Maven build the war file
  // Do not run tests in this step
  stage('Build war') {
    // TBD
	sh "${mvnCmd} clean package -DskipTests "
  }

  // Using Maven run the unit tests
//  stage('Unit Tests') {
//    // TBD
//	sh "${mvnCmd} test"
//  }

  // Using Maven call SonarQube for Code Analysis
//  stage('Code Analysis') {
//    // TBD
//	sh "${mvnCmd} sonar:sonar -s ./nexus_settings.xml -Dsonar.host.url=${sonarUrl}"
//  }

  // Publish the latest war file to Nexus. This needs to go into <nexusurl>/repository/releases.
  // Using the properties from the pom.xml file construct a filename that includes the version number from the pom.xml file
  // Also update the Gogs project openshift-tasks-ocp by editing the .s2i/environment file. This file needs to have a line
  //   WAR_FILE_LOCATION=<actual URL of the war file in nexus>
  // It is also a good idea to add another line like "BUILD_NUMBER=${BUILD_NUMBER}" to the environment file. Otherwise the
  // push to Gig/Gogs will fail in case the version number didn't change. ${BUILD_NUMBER} is one of the Jenkins built-in
  // variables.
  stage('Publish to Nexus') {
    // TBD
	sh "${mvnCmd} deploy -DskipTests=true -DaltDeploymentRepository=nexus::default::${nexusUrl}"
  }
  stage('Update s2i repo'){
  	def String warFileName     = "${groupId}.${artifactId}"
    	warFileName                = warFileName.replace('.', '/')

    	def String fullWarFilePath = "${nexusUrl}/${warFileName}/${version}/${artifactId}-${version}.war"

        git url: "http://moss:gogs@gogs-moss-cicd.apps.advdev.openshift.opentlc.com/moss/openshift-tasks-ocp.git"
	
        sh "echo WAR_FILE_LOCATION=${fullWarFilePath} > .s2i/environment"
        sh "echo BUILD_NUMBER=${BUILD_NUMBER}         >>.s2i/environment"
        
        // Update the Git/Gogs repository with the latest file
        def commit = "Release " + version
        sh "git config --global user.email moss@example.opentlc.com"
	sh "git config --global user.name moss"
	sh "git config --global user.password gogs"
	sh "git config --global -l"
        sh "git add .s2i/environment"
	sh "git commit -m \"${commit}\""
	sh "git push origin master"

  }

  // Build the OpenShift Image in OpenShift. Make sure to use the build configuration pointing to openshift-tasks-ocp
  // for the .s2i/bin/assemble script to retrieve the war file from the location in the .s2i/environment file.
  // Also tag the image with "TestingCandidate-${version}" - e.g. TestingCandidate-1.5
  stage('Build OpenShift Image') {
    // TBD
	def newTag = "devel"
	echo "New Tag: ${newTag}"

	openshiftBuild bldCfg: 'tasks', checkForTriggeredDeployments: 'false',
               namespace: 'moss-tasks-devel', showBuildLogs: 'false',
               verbose: 'false', waitTime: '', waitUnit: 'sec'

	openshiftVerifyBuild bldCfg: 'tasks', checkForTriggeredDeployments: 'false',
                     namespace: 'moss-tasks-devel', verbose: 'false', waitTime: ''

	openshiftTag alias: 'false', destStream: 'tasks', destTag: "${newTag}",
             destinationNamespace: 'moss-tasks-devel', namespace: 'moss-tasks-devel',
             srcStream: 'tasks', srcTag: 'latest', verbose: 'false'

  }

  // Deploy the built image to the Development Environment. Pay close attention to WHICH image you are deploying.
  // Make sure it is the one you just tagged in the previous step. You may need to patch the deployment configuration
  // of your application.
  stage('Deploy to Dev') {
    // TBD
	// DEploy
	openshiftDeploy depCfg: 'tasks', namespace: 'moss-tasks-devel'

  }

  // Run some integration tests (see the openshift-tasks Github Repository README.md for ideas).
  // Once the tests succeed tag the image as ProdReady-${version}
  stage('Integration Test') {
//	sh 'curl -i -u \'redhat:redhat1!\' -H "Content-Length: 0" -X POST http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks/task1'
//	sh 'curl -i -u \'redhat:redhat1!\' -H "Content-Length: 0" -X POST http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks/task2'
//	sh 'curl -u \'redhat:redhat1!\' -H "Accept: application/json" -X GET http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks/task1'
//	sh 'curl -u \'redhat:redhat1!\' -H "Accept: application/json" -X GET http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks/task2'
//	sh 'curl -u \'redhat:redhat1!\' -H "Accept: application/json" -X GET http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks'
//	sh 'curl -i -u \'redhat:redhat1!\' -H "Content-Length: 0" -X DELETE http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks/task1'
//	sh 'curl -i -u \'redhat:redhat1!\' -H "Content-Length: 0" -X DELETE http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks/task2'
//	sh 'curl -u \'redhat:redhat1!\' -H "Accept: application/json" -X GET http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/ws/tasks'
//
//	sh 'curl -u \'redhat:redhat1!\' -H "Accept: application/json" -X GET http://tasks-moss-tasks-devel.apps.advdev.openshift.opentlc.com/demo/load/5'
    // TBD
  }

  // Blue/Green Deployment into Production
  // -------------------------------------
  // Next two stages could be one.
  // Make sure to deploy the right version. If green is active then deploy blue, and vice versa.
  // You will need to figure out which application is active and set the target to the other.
  stage('Prep Production Deployment') {
    // TBD
	// Tag the devel latest to prod-ready if it passes integration test
	// 
	openshiftTag alias: 'false', destStream: 'tasks', destTag: "prod-ready",
             destinationNamespace: 'moss-tasks-devel', namespace: 'moss-tasks-devel',
             srcStream: 'tasks', srcTag: 'devel', verbose: 'false'
	
  }
  // Deploy the ProdReady-${version} image. Make sure this is the actual tagged image deployed!
  // Do not activate the new version yet.
  stage('Deploy new Version') {
    // TBD
	sh 'oc project moss-tasks-prod'
	//Get the currently routed deployment
	sh 'oc get routes/tasks --template={{.spec.to.name}} -n moss-tasks-prod > running'
	def running = readFile "${env.WORKSPACE}/running"
	echo "Currently routed service: ${running}"

	def newTarget = getNewTarget()

    // Trigger a new deployment
	openshiftDeploy deploymentConfig: "${newTarget}", namespace: 'moss-tasks-prod'
	openshiftVerifyDeployment deploymentConfig: "${newTarget}", namespace: 'moss-tasks-prod'

  }

  // Once approved (input step) switch production over to the new version.
  stage('Switch over to new Version') {
    input "Switch Production?"
	// Patch the route
	sh "oc patch -n moss-tasks-prod route/tasks --patch '{\"spec\":{\"to\":{\"name\":\"${newTarget}\"}}}'"

	
    // TBD
  }
}

// Convenience Functions to read variables from the pom.xml
// Do not change anything below this line.
def getVersionFromPom(pom) {
  def matcher = readFile(pom) =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}
def getGroupIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<groupId>(.+)</groupId>'
  matcher ? matcher[0][1] : null
}
def getArtifactIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<artifactId>(.+)</artifactId>'
  matcher ? matcher[0][1] : null
}
def colorSwitch(color) {
	def newRouteTarget=""
	if ($color == 'tasks-green'){
		newRouteTarget='tasks-blue'	
	}
	if($color == 'tasks-blue'){
		newRouteTarget='tasks-green'
	}
	return newRouteTarget	
}
def getCurrentTarget() {
  def currentTarget = readFile 'running'
  return currentTarget
}
def getNewTarget() {
  def currentTarget = getCurrentTarget()
  def newTarget = ""
  if (currentTarget == 'tasks-blue') {
      newTarget = 'tasks-green'
  } else if (currentTarget == 'tasks-green') {
      newTarget = 'tasks-blue'
  } else {
    echo "OOPS, wrong target"
  }
  return newTarget
}
