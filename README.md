# Continuous Deployment of Wildfly Applications using Developer Cloud and Application Container Cloud #

Continuing my theme of articles on Continuous Deployment (see [Dropbox](https://wbrianleonard.wordpress.com/2016/08/05/continuous-delivery-of-microservices-using-dropwizard-developer-cloud-and-application-container-cloud/), [Spring Boot](https://wbrianleonard.wordpress.com/2016/10/14/continuous-deployment-of-spring-boot-applications-using-developer-cloud-and-application-container-cloud/)) we're going to look at how [Wildfly](http://wildfly.org/) applications can be configured for CD.

As with my other examples, I'm going with the most basic Wildfly application, Hello World. This allows us to focus on the application changes that are required without getting distracted by the unrelated complexities of a larger application. Theoretically these steps should apply to any Wildfly application (possibly a topic of a future article).

Wildfly has an excellent repository of [quickstart](https://github.com/wildfly/quickstart) applications, of which one is [helloworld](https://github.com/wildfly/quickstart/tree/10.x/helloworld). This repository is a clone of the helloworld application with the changes necessary for Continuous Deployment to the [Oracle Application Container Cloud Service](https://cloud.oracle.com/en_US/acc?tabID=1447830811338). Those changes are:

## 1. Generate the Uber Jar ##
We use [Wildfly Swarm](http://wildfly-swarm.io/) to package the application into an Uber Jar. This was achieved by adding the following build plugin to the [pom.xml](https://github.com/wbleonard/accs-wildfly/blob/master/pom.xml):

			<!-- Wildfly Swarm plugin to create Uber JAR -->
			<plugin>  
                <groupId>org.wildfly.swarm</groupId>  
                <artifactId>wildfly-swarm-plugin</artifactId>  
                <version>1.0.0.Final</version>  
                <executions>  
                    <execution>  
                        <goals>  
                            <goal>package</goal>  
                        </goals>  
                    </execution>  
                </executions>  
			</plugin>  		

## 2. Configure for Deployment to the Application Container Cloud Service

### manifest.json ###

At a minimum we need to inform ACCS how to start the service, as well as the version of Java we want to use. This is specified in a metadata file called manifest.json. For other optional settings, see [Creating Metadata Files](http://docs.oracle.com/cloud/latest/apaas_gs/DVCJV/GUID-D98FB882-5E58-4318-9DCB-4B404FD86E14.htm#DVCJV-GUID-D98FB882-5E58-4318-9DCB-4B404FD86E14).

I created a new [manifest.json](https://github.com/wbleonard/accs-wildfly/blob/master/manifest.json) with the following contents. 

    {  
        "runtime": {  
            "majorVersion": "8"  
        },  
        "command": "java -jar -Dswarm.https.port=$PORT -Dswarm.context.path=helloworld target/wildfly-helloworld-swarm.jar"
	} 


Although the default port of Wildfly and Application Container Cloud is 8080, both are subject to change so it's best of instruct Wildfly which port to use, which we do via the PORT environment variable. See [Design Considerations](http://docs.oracle.com/cloud/latest/apaas_gs/DVCJV/GUID-06172FD2-778D-4882-9BE9-0C1ED9484E8E.htm#DVCJV-GUID-06172FD2-778D-4882-9BE9-0C1ED9484E8E) for more details.

### Deployment Archive ###
 
The application JAR file and the manifest.json file need to be packaged into a zip file for deployment to ACCS. I used the [Maven Assembly Plugin](http://maven.apache.org/plugins/maven-assembly-plugin/) to accomplish this task. The files here are essentially boiler plate code, identical to the files I used for the [Spring Boot](https://wbrianleonard.wordpress.com/2016/10/14/continuous-deployment-of-spring-boot-applications-using-developer-cloud-and-application-container-cloud/) application. For review:

I created a new file named [accs.xml](https://github.com/wbleonard/accs-wildfly/blob/master/src/assembly/accs.xml) in the src/assembly folder. 

	<?xml version="1.0" encoding="UTF-8"?>
	<assembly>
	    <id>ACCS</id>
	    <formats>
	        <format>zip</format>
	    </formats>
	    <includeBaseDirectory>false</includeBaseDirectory>
	    <files>
	        <file>
	            <source>manifest.json</source>
	            <outputDirectory>/</outputDirectory>
	        </file>
	    </files>
	    <fileSets>
	        <fileSet>
	            <directory>target</directory>
	            <outputDirectory>target</outputDirectory>
	            <includes>
	                <include>*.jar</include>
	            </includes>
	        </fileSet>
	    </fileSets>
	</assembly>

Then I added the Maven Assembly build plugin to the [pom.xml](https://github.com/wbleonard/accs-wildfly/blob/master/pom.xml):

            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                  <descriptor>src/assembly/accs.xml</descriptor>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>                
            </plugin>

## 3. Configure for Continuous Delivery ##
Other than the names of the Build Job and Deployment Configuration, these steps are identical to those I created for the for the [Spring Boot](https://wbrianleonard.wordpress.com/2016/10/14/continuous-deployment-of-spring-boot-applications-using-developer-cloud-and-application-container-cloud/) application.

## 4. Use ##

The application, although boring, is now running at [https://wildflyhelloworld-gse00001975.apaas.em2.oraclecloud.com/helloworld/HelloWorld](https://wildflyhelloworld-gse00001975.apaas.em2.oraclecloud.com/helloworld/HelloWorld):

![](https://github.com/wbleonard/accs-wildfly/blob/master/images/HelloWorld.JPG)

As with my previous examples, any new commits to the repository will trigger a build and, if successful, a deploy. 

## References ##

- [Part I: Building ‘right sized’ Java EE applications with Oracle Application Container Cloud and Wildfly Swarm](https://community.oracle.com/community/cloud_computing/oracle-cloud-developer-solutions/blog/2016/08/31/part-i-building-right-sized-java-ee-applications-with-oracle-application-container-cloud-and-wildfly-swarm) 
