# Continuous Deployment of Wildfly Applications using Developer Cloud and Application Container Cloud #

Continuing my theme of articles on Continuous Deployment (see [Dropbox](https://wbrianleonard.wordpress.com/2016/08/05/continuous-delivery-of-microservices-using-dropwizard-developer-cloud-and-application-container-cloud/), [Spring Boot](https://wbrianleonard.wordpress.com/2016/10/14/continuous-deployment-of-spring-boot-applications-using-developer-cloud-and-application-container-cloud/)) we're going to look at how [Wildfly](http://wildfly.org/) applications can be configured for CD.

As with my other examples, I'm going with the most basic Wildfly application, Hello World. This allows us to focus on the application changes that are required without getting distracted by the unrelated complexities of a larger application. Theoretically these steps should apply to any Wildfly application and maybe we will prove that in a later article.

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
## 2. Package for Application Container Cloud
