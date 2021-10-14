<h1>Weekly assesment 2 CI/CD Tools</h1>
<h2>1. Create Jenkins VM with internet access </h2>
<ul>
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
</ul>
<h2>2. Create Agent VM </h2>
<h2>3. Configure tools – NodeJS</h2>
<h2>4. Create “Multibranch Pipeline” pipeline job</h2>
<h2>5. Setup the GitHub webhook to trigger the jobs</h2>
<h2>6. Use Scripted pipeline instead of declarative</h2>
<h2>7. Spin up VM with installed</h2>
<h2>8. Add new stage for publishing artifacts into Artifactory</h2>
