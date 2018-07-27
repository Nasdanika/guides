# Single Site Eclipse Workspace

This guide explains how to build and deploy a "workspace" - a set of related projects - using [Tycho](https://www.eclipse.org/tycho/)/Maven and featuring a single site with a [P2](https://www.eclipse.org/equinox/p2/) repository and product binaries for [Eclipse products](https://wiki.eclipse.org/FAQ_What_is_an_Eclipse_product%3F]. 

A workspace includes:

* Bundle projects.
* Feature project(s).
* Parent project. You can also have an aggregator project, but it is not covered here.
* Repository project.

This guide assumes that you already know how to [build plug-ins, OSGi bundles, and Eclipse applications with Tycho](http://www.vogella.com/tutorials/EclipseTycho/article.html) and focuses on the part of building a [Maven site](https://maven.apache.org/plugins/maven-site-plugin/) and deploying the site along with the sources, p2 repository, and an Eclipse product if you have one.

## Parent project

Add the following plug-in to the ``pom.xml``:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-site-plugin</artifactId>
    <version>3.7.1</version>
    <configuration>
        <skip>true</skip>
        <skipDeploy>true</skipDeploy>
    </configuration>
</plugin>		
``` 

It tells Maven to skip site building and deploying for all plug-ins unless overridden.

## Repository project

Create a site under ``src\site`` - see [Maven site](https://maven.apache.org/plugins/maven-site-plugin/) for more details.

Add

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-site-plugin</artifactId>
    <version>3.7.1</version>
    <configuration>
        <skip>false</skip>
        <skipDeploy>false</skipDeploy>
    </configuration>
</plugin>		
```

under ``build/plugins``. It overrides parent settings.

Add

```
<extensions>
	<!-- Enabling the use of FTP -->
	<extension>
		<groupId>org.apache.maven.wagon</groupId>
		<artifactId>wagon-ftp</artifactId>
		<version>3.0.0</version>
	</extension>
</extensions>
```
 under ``build``. The above snippet uses FTP for deployment. You can use some other protocol.
 
 Copy repository and other resources to the site using [maven-resources-plugin](https://maven.apache.org/plugins/maven-resources-plugin/). The below snippet shows copying the repository and models and API documentation from other bundles.
 
 ```xml
			<plugin>
				<artifactId>maven-resources-plugin</artifactId>
				<version>3.0.1</version>
				<executions>
					<execution>
						<id>copy-repository</id>
						<phase>pre-site</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${basedir}/target/site</outputDirectory>
							<resources>
								<resource>
									<directory>${basedir}/target</directory>
									<includes>
										<include>repository/**</include>
										<include>org.nasdanika.codegen.repository-*.zip</include>
									</includes>
									<filtering>false</filtering>
								</resource>
							</resources>
						</configuration>
					</execution>
					<execution>
						<id>copy-modeldoc</id>
						<phase>pre-site</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${basedir}/target/site/modeldoc</outputDirectory>
							<resources>
								<resource>
									<directory>${basedir}/../org.nasdanika.codegen.editor/doc/site</directory>
									<filtering>false</filtering>
								</resource>								
							</resources>
						</configuration>
					</execution>
					<execution>
						<id>copy-model-javadoc</id>
						<phase>pre-site</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${basedir}/target/site/apidocs/model</outputDirectory>
							<resources>
								<resource>
									<directory>${basedir}/../org.nasdanika.codegen/target/site/apidocs</directory>
									<filtering>false</filtering>
								</resource>								
							</resources>
						</configuration>
					</execution>
					<execution>
						<id>copy-edit-javadoc</id>
						<phase>pre-site</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${basedir}/target/site/apidocs/edit</outputDirectory>
							<resources>
								<resource>
									<directory>${basedir}/../org.nasdanika.codegen.edit/target/site/apidocs</directory>
									<filtering>false</filtering>
								</resource>								
							</resources>
						</configuration>
					</execution>
					<execution>
						<id>copy-editor-javadoc</id>
						<phase>pre-site</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${basedir}/target/site/apidocs/editor</outputDirectory>
							<resources>
								<resource>
									<directory>${basedir}/../org.nasdanika.codegen.editor/target/site/apidocs</directory>
									<filtering>false</filtering>
								</resource>								
							</resources>
						</configuration>
					</execution>					
				</executions>
			</plugin>
``` 

Add distribution management and URL as shown in the example below:

```xml
<url>https://www.nasdanika.org/products/codegen/</url>
<distributionManagement>
	<site>
		<id>nasdanika-org</id>
		<url>ftp://${env.FTP_SERVER}/codegen</url>
	</site>
</distributionManagement>	
```

## Deploying sources

If you want to deploy sources of the workspace to the site follow the steps below.

### Parent project

Add 

```xml
<plugin>
	<artifactId>maven-assembly-plugin</artifactId>
	<version>3.1.0</version>
	<configuration>
	    <skipAssembly>true</skipAssembly>
	</configuration>
</plugin>			
```

It tells Maven to skip assembly unless overridden.

### Repository project

Add an assembly plugin definition following the example below.

```
<plugin>
	<artifactId>maven-assembly-plugin</artifactId>
	<version>3.1.0</version>
	<configuration>
		<skipAssembly>false</skipAssembly>
		<outputDirectory>${project.build.directory}/site</outputDirectory>
		<formats>zip</formats>
		<finalName>server</finalName>
		<appendAssemblyId>false</appendAssemblyId>
		<descriptors>
			<descriptor>src/assembly/workspace.xml</descriptor>
		</descriptors>
	</configuration>
        <executions>
          <execution>
            <id>create-archive</id>
            <phase>pre-site</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
</plugin>			
```

Create ``src\assembly\workspace.xml`` similar to the one shown below:

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.0.0 http://maven.apache.org/xsd/assembly-2.0.0.xsd">
  <id>project</id>
  <formats>
    <format>tar.gz</format>
    <format>tar.bz2</format>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${project.basedir}/..</directory>
      <outputDirectory>/</outputDirectory>
      <useDefaultExcludes>false</useDefaultExcludes>
      <excludes>
        <exclude>*/target/**</exclude>
        <exclude>*.jar</exclude>
        <exclude>.git/**</exclude>
        <exclude>.circleci/**</exclude>
        <exclude>ftp-clean.xml</exclude>
      </excludes>
    </fileSet>
  </fileSets>
</assembly>
```

## Deployment

To build and deploy use the following command:
``mvn clean javadoc:javadoc package site-deploy``. Make sure that you have your server user name and password defined in ``settings.xml``.

### CircleCI

To build and deploy from [CircleCI](https://circleci.com) follow the steps below.

Create ``.circleci`` directory with two files - ``config.yml`` and ``settings.xml``. 

``config.yml`` example:

```yaml
# Java Maven CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-java/ for more details 
#
version: 2
jobs:
  build:
    docker:
      # specify the version you desire here
      - image: circleci/openjdk:8-jdk
      
      # Specify service dependencies here if necessary
      # CircleCI maintains a library of pre-built images
      # documented at https://circleci.com/docs/2.0/circleci-images/
      # - image: circleci/postgres:9.4

    working_directory: ~/repo

    environment:
      # Customize the JVM maximum heap limit
      MAVEN_OPTS: -Xmx3200m
    
    steps:
      - checkout
      - run: mvn -s .circleci/settings.xml clean site-deploy
```

``settings.xml`` example:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nasdanika-org</id>
      <username>${env.FTP_USER}</username>
      <password>${env.FTP_PASSWORD}</password>
    </server>
  </servers>
</settings>
```

Then define ``FTP_SERVER``, ``FTP_USER``, and ``FTP_PASSWORD`` environment variables in CircleCI and run a build.

### Cleaning the remote directory

Maven Wagon does not clean the remote folder before deployment - it just deploys over. To clean the remove folder you can use [maven-antrun-plugin](http://maven.apache.org/plugins/maven-antrun-plugin/) as shown below:

```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-antrun-plugin</artifactId>
	<version>1.8</version>
	<executions>
		<execution>
			<id>ftp</id>
			<phase>post-site</phase>
			<configuration>
				<target>
					<ftp passive="yes" action="delete" server="${env.FTP_SERVER}" userid="${env.FTP_USER}" password="${env.FTP_PASSWORD}" remotedir="/site">
						<fileset defaultexcludes="false">
							<include name="**" />
						</fileset>
					</ftp>
					
					<ftp passive="yes" action="rmdir" server="${env.FTP_SERVER}" userid="${env.FTP_USER}" password="${env.FTP_PASSWORD}" remotedir="/site">
						<fileset defaultexcludes="false">
							<include name="**" />
						</fileset>
					</ftp>
				</target>
			</configuration>
			<goals>
				<goal>run</goal>
			</goals>
		</execution>
	</executions>
	<dependencies>
		<dependency>
			<groupId>commons-net</groupId>
			<artifactId>commons-net</artifactId>
			<version>1.4.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.ant</groupId>
			<artifactId>ant-commons-net</artifactId>
			<version>1.8.1</version>
		</dependency>
	</dependencies>
</plugin>			
```



