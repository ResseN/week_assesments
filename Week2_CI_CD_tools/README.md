<h1>Weekly assesment 2 CI/CD Tools</h1>
<h2>1. Create Jenkins VM with internet access </h2>
<ul>
  <li>sudo -i</li>
  <li>apt update</li>
  <li>apt upgrade</li>
  <li>apt-get install openjdk-8-jdk git</li>
  <li>wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -</li>
  <li>sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'</li>
  <li>sudo apt-get update</li>
  <li>sudo apt-get install jenkins</li> 
  <li>systemctl enable jenkins</li>
  <li>sed -i 's/8080/8081/g' /etc/default/jenkins</li>
  <li>systemctl restart jenkins.service</li>
  <li>su - jenkins</li>
  <li>ssh-keygen</li>
  <li>cat .ssh/id_rsa.pub</li>
  <li>ssh root@18.157.187.4</li>
  <blockquote>
   <pre> 
    The authenticity of host '18.157.187.4 (18.157.187.4)' can't be established.
    ECDSA key fingerprint is SHA256:Wyjqy066e3md7DzTSRwmUZWMrIWcbNUHSdZyPZRK/Ds.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '18.157.187.4' (ECDSA) to the list of known hosts.
  </pre>
  </blockquote>
</ul>
<p>Enable role-based policy</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/enable_role_based_strategy.png" height="600px"/> 
<p>Create role for user</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/manage_roles.png" height="600px"/> 
<p>Assign role for jenkins-yuriyrurenko to user</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/assign_roles.png" height="600px"/> 
<h2>2. Create Agent VM </h2>
<ul>
  <li>sudo -i</li>
  <li>apt update</li>
  <li>apt upgrade</li>
  <li>apt install git openjdk-8-jre</li>
  <li>mkdir /opt/jenkins</li>
  <li>nano .ssh/authorized_keys # insert copied id_rsa.pub from jenkins host</li>
</ul>
<p>Add jenkins node</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Jenkins_node_add.png" height="600px"/> 
<p>Check jenkins node status</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/jenkins_node_status.png" height="200px"/> 
  <h2>3. Configure tools – NodeJS</h2>
<p>Manage Jenkins -> Manage plagins -> Available Plagins -> NodeJS Plagin -> Install & restart jenkins</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Plagin_install.png" height="600px"/> 
<p>Manage Jenkins -> Global Tools Configuration -> NodeJS -> Add NodeJS</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Nodejs_installation.png" height="600px"/> 
<h2>4. Create “Multibranch Pipeline” pipeline job</h2>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Create%20Lab_name%20folder_multibranch_pipeline.png" height="600px"/> 
<p>Config multibranch pipeline</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Multibranch_pipeline_config.png" height="600px"/> 
<p>Use fork from previous week_assessment <a href="https://github.com/ResseN/mdt">https://github.com/ResseN/mdt</a></p>
<p>Write <a href=https://github.com/ResseN/mdt/blob/master/Jenkinsfile>Jenkinsfile</a></p>
<blockquote>
   <pre>
   pipeline{
    agent {
        label 'ubuntu_2004_agent'
    }
    stages{
        stage("Compressing"){
            parallel{
                stage("JS"){
                    steps{
                        nodejs(nodeJSInstallationName: 'NodeJS16'){
                            sh 'find www/js/ -name "*.js" -printf \'%f\n\'| xargs -I {} uglifyjs www/js/{} -o www/min/min.{}'
                        }
                    }
                }
                stage("CSS"){
                    steps{
                        nodejs(nodeJSInstallationName: 'NodeJS16'){
                            sh 'find www/css/ -name "*.css" -printf \'%f\n\'| xargs -I {} cleancss www/css/{} -o www/min/min.{}'
                        }
                    }
                }
            }
        }
        stage("Archieving"){
            steps{
                sh 'tar cf mdt.tar --exclude=.git* --exclude=www/css --exclude=www/js --exclude=mdt.tar . '
                archiveArtifacts artifacts: 'mdt.tar', allowEmptyArchive: false, fingerprint: true, onlyIfSuccessful: true
            }
        }
    }
}
   </pre>
</blockquote>
<p>Job log in <a href=https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Job.log>log</a> file</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Stage_log.png" height="600px"/> 
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Stage_log_blue_ocean.png" height="250px"/> 

<h2>5. Setup the GitHub webhook to trigger the jobs</h2>
<p>To setup Jenkins for trigger the job - install plagin</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Trigger_plagin_install.png" height="100%"/>
<p>After installing plagin in multibranch pipeline job in section "Scan Repository Triggers" choose Periodically and set interval for Poll SCM</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/trigger%20poll%20scm.png" height="100%"/>
<p>Scanning repository will performed every set interval and if changes would found in repository - job started automatically</p>
<p>To setup job to been triggered by github webhook must in "Scan Repository Triggers" choose "scan by webhook" and generate some token</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/trigger_webhook.png" height="100%"/>
<p>On github side in project repository enter to Settings -> Webhooks -> Add webhook -> http://18.192.207.233:8081/multibranch-webhook-trigger/invoke?token=triggertokenforwebhook</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/github_webhook_create.png" height="100%"/>
<p>Check deliverability of webhooks to Jenkins</p>
<img src="https://github.com/ResseN/week_assesments/blob/main/Week2_CI_CD_tools/Github_webhook_log_.png" height="100%"/>

<h2>6. Use Scripted pipeline instead of declarative</h2>
<h2>7. Spin up VM with installed Artifactory</h2>
<ul>
  <li>apt update</li>
  <li>apt upgrade</li>
  <li>lsb_release -c</li>
  <li>wget -qO - https://releases.jfrog.io/artifactory/api/gpg/key/public | sudo apt-key add -</li>
  <li>echo "deb https://releases.jfrog.io/artifactory/artifactory-pro-debs focal main" | sudo tee -a /etc/apt/sources.list</li>
  <li>sudo apt-get update && sudo apt-get install jfrog-artifactory-pro</li>
  <li>systemctl start artifactory.service</li>
  <li>systemctl enable artifactory.service</li>
</ul> 
<h2>8. Add new stage for publishing artifacts into Artifactory</h2>
