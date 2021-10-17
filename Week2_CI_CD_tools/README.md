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

<blockquote>
   <pre>
   Console Output
Branch indexing
09:14:07 Connecting to https://api.github.com with no credentials, anonymous access
Obtained Jenkinsfile from e1e28e61819d34abd31e66873af28ecbb049688d
[Pipeline] Start of Pipeline
[Pipeline] node
Running on ubuntu_2004_agent in /opt/jenkins/workspace/yRurenko_week_assesment_2_master
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning with configured refspecs honoured and without tags
Cloning repository https://github.com/ResseN/mdt.git
 > git init /opt/jenkins/workspace/yRurenko_week_assesment_2_master # timeout=10
Fetching upstream changes from https://github.com/ResseN/mdt.git
 > git --version # timeout=10
 > git --version # 'git version 2.25.1'
 > git fetch --no-tags --force --progress -- https://github.com/ResseN/mdt.git +refs/heads/master:refs/remotes/origin/master # timeout=10
Avoid second fetch
Checking out Revision e1e28e61819d34abd31e66873af28ecbb049688d (master)
Commit message: "Merge pull request #2 from ResseN/feature_archive"
First time build. Skipping changelog.
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Compressing)
[Pipeline] parallel
[Pipeline] { (Branch: JS)
[Pipeline] { (Branch: CSS)
[Pipeline] stage
[Pipeline] { (JS)
[Pipeline] stage
[Pipeline] { (CSS)
[Pipeline] nodejs
[Pipeline] nodejs
[Pipeline] {
[Pipeline] sh
[Pipeline] {
[Pipeline] sh
+ xargs -I {} uglifyjs www/js/{} -o www/min/min.{}
+ find www/js/ -name *.js -printf %f

+ xargs -I {} cleancss www/css/{} -o www/min/min.{}
+ find www/css/ -name *.css -printf %f

 > git config remote.origin.url https://github.com/ResseN/mdt.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/master:refs/remotes/origin/master # timeout=10
 > git config core.sparsecheckout # timeout=10
 > git checkout -f e1e28e61819d34abd31e66873af28ecbb049688d # timeout=10
[Pipeline] }
[Pipeline] // nodejs
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] }
[Pipeline] // nodejs
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // parallel
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Archieving)
[Pipeline] sh
+ tar cf mdt.tar --exclude=.git* --exclude=www/css --exclude=www/js --exclude=mdt.tar .
[Pipeline] archiveArtifacts
Archiving artifacts
Recording fingerprints
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
   </pre>
</blockquote>
<h2>5. Setup the GitHub webhook to trigger the jobs</h2>
<h2>6. Use Scripted pipeline instead of declarative</h2>
<h2>7. Spin up VM with installed Artifactory</h2>
<h2>8. Add new stage for publishing artifacts into Artifactory</h2>
