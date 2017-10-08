---
layout: post
authors: [bart_blommaerts]
title: "Continuous Deployment with LiveRebel"
tags: [continuous deployment, liverebel]
category: Java
comments: true
---

At <a title="Devoxx" href="http://www.devoxx.com/" target="_blank">Devoxx</a>, I had an interesting chat with the guys from <a title="ZeroTurnaround" href="http://zeroturnaround.com/" target="_blank">ZeroTurnaround</a> on their <a title="LiveRebel" href="http://zeroturnaround.com/software/liverebel/" target="_blank">LiveRebel</a> product. LiveRebel is a Java EE server management tool, designed for making automated, instant updates to Java applications in production without server downtime. ZeroTurnaround had prepared a nice demo and I was really eager to try it myself .. on a real project.

One of the projects I'm currently working on, uses <a title="Jenkins" href="http://jenkins-ci.org/" target="_blank">Jenkins</a> as a CI server. Our applications are built in <a title="Maven" href="http://maven.apache.org/" target="_blank">Maven</a> and use <a title="Nexus" href="http://www.sonatype.org/nexus/" target="_blank">Sonatype Nexus</a> as a central repository. Post-build, we have <a title="Sonar" href="http://www.sonarsource.org/" target="_blank">Sonar</a> integration. As I didn't want my proof-of-concept to interact with our actual application server, I installed a new <a title="GlassFish" href="http://glassfish.java.net/" target="_blank">GlassFish 3.1.2</a> server. Just to make sure the application server didn't interfere with the result, I manually deployed the WAR of the selected application first on the new GlassFish server. That part didn't pose an issue.

After downloading the stable version of LiveRebel, I started the LiveRebel Command Center:
<pre>./lr-command-center.sh run</pre>
<div>&nbsp;</div>
This results in a web front-end that can be used to easily configure servers and applications:

<p style="text-align: center;">  
  <img class="image fit" alt="liverebel" src="/img/older/liverebel1.png">
</p>

On that front-end, you need to select "Add Server" and select the platform you want to deploy on (GlassFish in my case). The web front-end will present an easy to follow guide on how to install the LiveRebel <a title="Java Agent" href="http://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html" target="_blank">Java Agent</a>: (lr-agent-installer.jar) for that specific platform. After restarting the GlassFish domain .. all is ready. It's also possible to enable/disable the LiveRebel java agent:
<pre>lr-agent/bin/admin.sh [enable | disable]</pre>
<div>&nbsp;</div>
which is interesting if you don't want LiveRebel to be intrusive, after deploying. In the front-end, you can also generate an authentication token in the "admin"-menu. This authentication token will be used in Jenkins.

The only adjustment I had to make in my application was adding the LiveRebel Maven plugin to the POM of the application. The LiveRebel Maven plugin, takes care of making the liverebel.xml file. The liverebel.xml configuration file is used by LiveRebel to manage the application versioning and should be present in every application archive, including deployed applications managed by LiveRebel and archives uploaded to the LiveRebel console.

<pre>
&lt;plugin&gt;
   &lt;groupId&gt;org.zeroturnaround&lt;/groupId&gt;
   &lt;artifactId&gt;jrebel-maven-plugin&lt;/artifactId&gt;
   &lt;!-- Optional configuration --&gt;
   &lt;configuration&gt;
      &lt;name&gt;${project.artifactId}-development&lt;/name&gt;
      &lt;version&gt;${project.version}-${maven.build.timestamp}&lt;/version&gt;
   &lt;/configuration&gt;
   &lt;executions&gt;
      &lt;execution&gt;
         &lt;id&gt;generate-liverebel-xml&lt;/id&gt;
         &lt;phase&gt;process-resources&lt;/phase&gt;
         &lt;goals&gt;
            &lt;goal&gt;generate-liverebel-xml&lt;/goal&gt;
         &lt;/goals&gt;
      &lt;/execution&gt;
   &lt;/executions&gt;
&lt;/plugin&gt;
</pre>

After installing LiveRebel, I downloaded the <a title="LiveRebel Deploy Plugin" href="https://wiki.jenkins-ci.org/display/JENKINS/LiveRebel+Deploy+Plugin" target="_blank">LiveRebel Deploy Plugin</a> for Jenkins. This plugin requires the LiveRebel authentication token. In my Project in Jenkins I added a "Post Step": Deploy or Update artifact with LiveRebel. After selecting my new server I fired up a Jenkins build ... And saw that my application was getting deployed to the new server .. and was working as expected.

I have to say that I was impressed with LiveRebel and how easy it was to get it all up and running with an actual application. Since it's started from Jenkins, a lot can be configured: deploy time, naming, .. The LiveRebel web front-end makes it possible for Operations to do automated deploys if continuous deployment isn't "possible/accepted" in a productive environment.

I made a small scheme to illustrate the final set-up of my proof-of-concept:

<p style="text-align: center;">  
  <img class="image fit" alt="liverebel" src="/img/older/ci2.png">
</p>


All of the steps now work automatically, without any manual actions. I haven't looked into more advanced features such as database integration, rolling restarts or the possibility to maintain user sessions over deploys, but I'm intrigued to play with it more.