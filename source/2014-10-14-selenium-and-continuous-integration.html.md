---
title: Selenium and Continuous Integration
date: 2014-10-14 17:03 UTC
tags: selenium, ci
---

Overview
--------
This article covers:

-   Why you would want to use continuous integration.
-   The main tools you'll use.
-   Some important strategies when testing.

We'll assume you are familiar with using a terminal or command prompt. 

Source code can be found on [Github](https://github.com/alexec/selenium-ci).

Why CI?
-------
One of the main ways you can improve the speed and quality of your testing is to run your tests on a CI server. This has a number of benefits to be gained by running them on a Continuous Integration server compared to  only running them manually on your own desktop machine.

1.  It frees up your desktop machine for writing code.
2.  You don't need to be told to run the test each time your application has a code change. The CI can build the application, deploy it and then run your tests automatically.
3.  Your CI can be set-up to run each test in different browsers, making cross-browser testing much easier.
4.  Your CI can also run each test in different languages, different time zones, and different times of day.
5.  Your CI can record information about each run and store it in database, so you can refer back at a later time to see just when a problem started occurring.
6.  You can configure your CI to run the tests on a regular basis, so that if anything changes, you'll get notified.
7.  If a problem occurs with a test, you don't need to tell the developer. The CI will notify you and your team automatically via email.

There's a number of different strategies for running tests, closely related to how you set-up your test environments. Each has a set of pros and cons. We'll talk a little bit about this, and we'll talk a little about some popular commercial and open source systems you can use.

Build Tools
-----------
A build tool is a piece of software that actually builds your application and typically packages it into a single binary file. There's a number build tools you can choose from, here are a couple of popular ones:

-  [Maven](http://maven.apache.org) is the most popular build tool on the JVM. It uses the
philosophy of "convention over configuration". This not only means it is easy to understand, it has excellent support and integration into IDEs and CI servers.
- [Ant](http://ant.apache.org) is Java's venerable build tool. It's still quite common.
- [Gradle](http://www.gradle.org) is a Groovy based build tool that is extremely flexible. It’s
DSL based system is a bit more esoteric that Maven, but it is popular for complex builds.

Binary Repository
-----------------
A binary repository is where you store your application once it's build. All have a common set of feature, such as security, the ability to search and query artefacts, and a web interface to manage it. Here are two you might consider:

- [Sonatype Nexus](http://www.sonatype.com/nexus) has both a supported commercial version and a free open source version.
- [Artifactory](http://www.jfrog.com/open-source/) also has commercial and open source versions.

CI Server
---------
Your CI server is the computer that actually calls the build tool to build your application. All CI servers tend support a common set of features such as executing builds either manually, on a schedule or whenever the source code changes; creating historical reports on your builds, storing the output from tests, and emailing or instant messaging you when there's a problem. Here are some popular CI servers:

-  [Jenkins](http://jenkins-ci.org) is an open source version of **Hudson**. It's extremely popular, is very well supported, and has a cornucopia of plugins. Some say
it is starting to show it ages these days, but I’m betting it has a lot of life in it.
-  [Go](http://www.thoughtworks.com/products/go-continuous-delivery) is a CI server created from Thoughtworks. It has both a commercial and open source version.
-  [TeamCity](https://www.jetbrains.com/teamcity) is a server from JetBrains.
-  [Bamboo](https://www.atlassian.com/software/bamboo) is a commercial tool from Atlassian, who are best known for their JIRA and Confluence applications. It's very well integrated into those.
-   [Travis CI](https://travis-ci.org) and [Drone.IO](https://drone.io) are both commercial/open source CI servers that by default run builds on isolated virtual machines.

Continuous Integration Test Strategies
--------------------------------------
In a typical strategy you'll want to do a number of basic steps:

1.  Create a binary build of the web application you are testing.
2.  Copy that build to a repository for safekeeping.
3.  Set-up your test system to a known initial state.
4.  Deploy your binary to your test system.
5.  Run your tests on your CI.
6.  Create a report and save it in your CI's database.
7.  Inform someone that the tests have been run.
8.  Optionally, clean-up.
9.  Potentially, deploy the binary to your production system.

You've got a few options on how you can run this. I imagine you're already using one of the great build tools such as Maven. What you need to decide on is where you're going to run your application and how you set-up up your test system.

Running your application under test
-----------------------------------
You can run your application in one of two places. The first is **locally** on the machine that runs the tests, and second is on a dedicated test system. Running tests on the local machine will simplify the set-up of the tests, they can always talk to localhost. It might be quite easy to initialise the local system to a known state, you don't need dedicated computers set-up ready for testing, and if you can run it on your local machines, you can always run your tests before you share your change.

However, you might be using a commercial database who's license agreement means you can't install it locally, or you might be unable to have some required service such on JNDI or JMS running. These mean you'll have to run your tests on a dedicated **remote** system means you'll need to set-up and maintain that system, for cost reasons you may have to share it with other teams, which might mean that initialising it into a known state might not be feasible.

If you want some guidance as to which option to use, I'd try and use the first to start with, as it's generally cheaper, and it's often quicker to determine if that's feasible and then move to a dedicate system if it is not, than vice-versa. 

### Setting up Maven to run your application locally

Let do an example of a local The Cargo plugin will start and stop a test system before and after your tests have run. Lets work through an example.

Create a Maven project using the web app archetype:

~~~
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-webapp -DarchetypeVersion=1.0 -DgroupId=sip -DartifactId=ci -Dversion=1.0.0-SNAPSHOT -B
~~~

Test that this will build by using:

~~~
mvn verify
~~~

Now add the Cargo plugin section to the build section of your `pom.xml`.

~~~xml
<plugins>
	<plugin>
		<groupId>org.codehaus.cargo</groupId>
		<artifactId>cargo-maven2-plugin</artifactId>
		<version>1.4.10</version>
	</plugin>
</plugins>
~~~

You can check this works using:

~~~
mvn cargo:run
~~~

This will start your server and you can check your application is running by opening <http://localhost:8080/ci>, you should see "Hello World!". Let's write a basic test using what we've already learnt. Add this to the `dependencies` section of the `pom.xml`:

~~~xml
<dependency>
	<groupId>org.seleniumhq.selenium</groupId>
	<artifactId>selenium-java</artifactId>
	<version>2.43.1</version>
	<scope>test</scope>
</dependency>
~~~

Let's use a later version of JUnit (it's got more features):

~~~xml
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.11</version>
	<scope>test</scope>
</dependency>
~~~

Let's create a test to make sure that "Hello Word!" is displayed on the home page. Create `src/test/java/ci/HomepageIT.java`

~~~java 
package ci;
import org.junit.After; 
import org.junit.Before; 
import org.junit.Test; 
import org.openqa.selenium.By; 
import org.openqa.selenium.WebDriver; 
import org.openqa.selenium.firefox.FirefoxDriver;
import static org.junit.Assert.assertEquals;

public class HomepageIT {

	private WebDriver driver;
	
	@Before public void setUp() throws Exception { driver = new FirefoxDriver(); }
	
	@After public void tearDown() throws Exception { driver.close(); }

	@Test public void testHomepage() throws Exception {
		driver.get("http://localhost:8080/ci");
		
		assertEquals("Hello World!", driver.findElement(By.tagName("h2")).getText()); 
	} 
} 
~~~

What we want to do is start and stop the server before and after running our integration tests. We often don't want to run the tests as part of the default build, as they can be time consuming. We'll add a profile to our build to run our just our integration tests:

~~~xml
<profiles>
	<profile>
		<id>run-its</id>
	</profile>
</profiles>
~~~

This will only be run if we activate it:

~~~
mvn verify -Prun-its
~~~

We can do this by adding the following lines within this new profile your `pom.xml`:

~~~xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.cargo</groupId>
            <artifactId>cargo-maven2-plugin</artifactId>
            <executions>
                <execution>
                    <id>start</id>
                    <phase>pre-integration-test</phase>
                    <goals>
                        <goal>start</goal>
                    </goals>
                </execution>
                <execution>
                    <id>stop</id>
                    <phase>post-integration-test</phase>
                    <goals>
                        <goal>stop</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
~~~

Maven finally needs to be told to run the tests, and we can do this using the failsafe plugin which you can add to the `plugins` section of your
`pom.xml`.

~~~xml
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <executions> 
        <execution> 
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal> 
            </goals>
        </execution> 
    </executions>
</plugin>
~~~

Now you can have Maven start your application, run your tests, and clean up after itself, with just one command:

~~~
mvn verify -Prun-its
~~~

You'll see this line:

~~~
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
~~~

### Setting up Maven to run your application remotely

Now lets work through a **remote** example.

If you're using a standard container such as Tomcat or JBoss, you can use the Cargo plugin for this as well. To run this example you'll need to do a bit extra work and install Tomcat locally. You can download Tomcat 8 it from <http://tomcat.apache.org/>. Un-zip the download and open up the directory that is unpacked. By default, users are not enabled, so we need to configure one. From within the Tomcat directory, open `conf/tomcat-users.xml`, and add these lines inside the `tomcat-users` element:

~~~xml 
<role rolename="manager-script"/>
<role rolename="manager-gui"/> 
<user username="admin" password="secret" roles="manager-script,manager-gui"/>
~~~

To start it up, open up a terminal, change directory into the unzipped directory and then and execute either `./bin/catalina.sh run` or `.\bin\catalina.bat
run` (if you are on Windows). You should see the following:

~~~
Server startup in 477 ms
~~~

You can verify that it's worked by logging in to <http://localhost:8080/manager/html> using the username and password above. Next, we can update our `pom.xml` with the details needed to deploy the application. Change the Cargo's to look like this:

~~~xml 
<plugin>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-maven2-plugin</artifactId>
    <executions> 
        <execution>
            <id>redeploy</id>
            <phase>pre-integration-test</phase> 
            <goals>
                <goal>redeploy</goal> 
            </goals>
        </execution> 
    </executions>
    <configuration> 
        <container>
            <containerId>tomcat8x</containerId>
            <type>remote</type> 
        </container>
        <configuration> 
            <type>runtime</type>
            <properties>
                <cargo.remote.username>admin</cargo.remote.username>
                <cargo.remote.password>secret</cargo.remote.password>
            </properties> 
        </configuration>
    </configuration>
</plugin>
~~~
 
We can ask Maven to build and deploy our application, and then run our tests, in the same one line.

~~~
mvn verify -Prun-its
~~~

Congratulations! You've now got a remote build ready to go!

How to set-up your tests
------------------------
You'll need to set-up your system in a known state. The ideal test is stateless - it requires no set-up before it's run, or tear-down afterwards: it's always in a known state. Unreliable or "flaky" tests are a massive maintenance problem. I'm sure you want to invest your time writing valuable new code rather than maintaining a test system. There's no better way to have unreliable tests that to have to hope that your test system is in a suitable state.

There's two ways you can achieve this.

-  The first way is to **initialise** your test system from scratch each time. For example drop the database, recreate it, and the load a set of test data into it. This is ideal if you don't have a complex system with lots of test data. 
- The second way is to try and **compensate** for the current state and move your test system from whatever state it is in, to the necessary state. This is a suitable approach if recreating the system is time-consuming, or if you need to use an existing system. The trade off is that the type of scripts you'll need to write might be time-consuming in themselves.

Let's consider this example. You want a test to make sure a user who has been disabled cannot login. You should not create this user in another test, because you'll be coupling your tests together (mean that many tests could fail but there may only be a single problem test), or it's simply not possible to disable users in your application.

If you're using initialisation strategy, at the start of the test run:

1.  Insert a test user named "disableduser" with a password "testpassword" into the database.
2.  Mark them as disabled.

If you're using the compensation strategy, you would:

1.  Find out if your user already exists.
2.  If they don't create them as before.
3.  If they do, make sure their password is set to "testpassword".
4.  Check to see if they have any differences to a newly register user that might affect your test, and correct them.
5.  Mark them as disabled.

Or you could:

1.  Find an unused username.
2.  Insert a test user with that username with a password "testpassword" into the database.
3.  Mark them as disabled.
4.  Store that username somewhere the test can reference it from.

You can see from this example that the compensation strategy has some problems. In the first option you'll could easily end up with a user that might not be able to login for some other reason. In the second you'll rapidly end up with far more users than you need for your tests, slowing your test system up.

Another important consideration is the design of your build, is that you should not rely on your tests to set themselves up.

 

TODO

Summary
-------

 

TODO
