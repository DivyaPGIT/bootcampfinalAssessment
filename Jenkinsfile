try{
    node{
        def mavenHome
        def mavenCMD
        def docker
        def dockerCMD
        def tagName = "hello"
        
        stage('Preparation'){
            echo "Preparing the Jenkins environment with required tools..."
            mavenHome = tool name: 'mavennew', type: 'maven'
            mavenCMD = "${mavenHome}/bin/mvn"
            docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
            dockerCMD = "$docker/bin/docker"
        }
        
        stage('git checkout'){
            echo "Checking out the code from git repository..."
            git 'https://github.com/DivyaPGIT/bootcampfinalAssessment.git'
        }
        
        stage('Build, Test and Package'){
            echo "Building the application..."
			sh "${mavenCMD} clean test"
            sh "${mavenCMD} clean package"
        }
        stage('Sonar Scan'){
            echo "Scanning application for vulnerabilities..."
            sh "${mavenCMD} sonar:sonar -Dsonar.host.url=http://34.126.170.251:9000/"
        }	
        stage('publish report'){
            echo " Publishing HTML report.."
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
        }	
        stage('Build Docker Image'){
            echo "Building docker image for hello application ..."
			sh "sudo apt -y install docker.io"
			sh "docker --version"
			sh "sudo chmod 777 /var/run/docker.sock"
            sh "${dockerCMD} build -t divyasashvi/newrepo:${tagName} ."
        }	
        stage("Push Docker Image to Docker Registry"){
            echo "Pushing image to Docker Hub"
			withCredentials([string(credentialsId: 'docpwd', variable: 'docpwd')]) {
			sh "${dockerCMD} login -u divyasashvi -p ${docpwd}"
            sh "${dockerCMD} push divyasashvi/newrepo:${tagName}"
   
			}
        }
        stage('Deploy Application'){
            echo "Installing desired software.."
            ansiblePlaybook credentialsId: 'sshprivate', disableHostKeyChecking: true, installation: 'ansible 2.9.22', inventory: '/etc/ansible/hosts', playbook: 'ansibleplaybook.yml'
			emailext body: 'Your build has been successfull', subject: 'Build Result', to: 'pdivyairtt@gmail.com'
        }


    }
}
catch(Exception err){
    echo "Exception occured..."
    currentBuild.result="FAILURE"
    emailext body: 'Your build has been failed', subject: 'Build Result', to: 'pdivyairtt@gmail.com'
}
finally {
    (currentBuild.result!= "ABORTED") && node("master") {
        echo "Sent email notification for every build"
        
    }
    
}
