# Eclipse Package

This guide explains how to automatically build and deploy a custom Eclipse package based on an existing Eclipse package.
This guide was created based on [Nasdanika Tool Suite](../../../tools/index.html), which can be used as a starting point/reference implementation.
The guide assumes that deployment is done over FTP.

The process of creation of a custom eclipse package includes the following steps:
* Create a feature which includes features and/or bundles which need to be installed into the package.
* Build and deploy a feature repository and site.
* Download the base eclipse package.
* Unzip.
* Execute [p2 director](http://help.eclipse.org/photon/index.jsp?topic=/org.eclipse.platform.doc.isv/guide/p2_director.html) to install the feature from the repository.
* Zip.
* Upload the package to the site.

An eclipse package workspace consists of three Eclipse/[Maven](https://maven.apache.org/)/[Tycho](https://www.eclipse.org/tycho/) projects:
* Parent
* Feature
* Repository

The repository has two pom files - one to build the P2 repository and the site and the other to build and upload the binary distribution.

## Parent 

The parent project contains ``pom.xml`` which:
* Defines plugins to be used by child projects.
* Contains a list of p2 repositories.
* Lists child modules.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
	xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.nasdanika.tools</groupId>
	<artifactId>org.nasdanika.tools.parent</artifactId>
	<version>0.1.0-SNAPSHOT</version>
	<packaging>pom</packaging>

	<!-- tycho requires maven >= 3.0 -->
	<prerequisites>
		<maven>3.0</maven>
	</prerequisites>

	<properties>
		<tycho-version>1.2.0</tycho-version>
	</properties>

	<repositories>
		<!-- configure p2 repository to resolve against -->
		<repository>
			<id>photon</id>
			<url>http://download.eclipse.org/releases/photon</url>
			<layout>p2</layout>
		</repository>
		<repository>
			<id>orbit</id>
				<url>http://download.eclipse.org/tools/orbit/downloads/drops/R20180606145124/repository</url>
			<layout>p2</layout>
		</repository>
		<repository>
			<id>tycho-configurator</id>
				<url>http://repo1.maven.org/maven2/.m2e/connectors/m2eclipse-tycho/0.8.1/N/0.8.1.201704211436/</url>
			<layout>p2</layout>
		</repository>
		<repository>
			<id>nasdanika</id>
			<url>https://www.nasdanika.org/products/site/repository</url>
			<layout>p2</layout>
		</repository>
		
	</repositories>
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
			    <artifactId>maven-site-plugin</artifactId>
			    <version>3.7.1</version>
			    <configuration>
			        <skip>true</skip>
			        <skipDeploy>true</skipDeploy>
			    </configuration>
			</plugin>		
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<version>3.1.0</version>
				<configuration>
				    <skipAssembly>true</skipAssembly>
				</configuration>
			</plugin>			
			<plugin>
				<groupId>org.eclipse.tycho.extras</groupId>
				<artifactId>tycho-document-bundle-plugin</artifactId>
				<version>${tycho-version}</version>
			</plugin>			
			<plugin>
				<!-- enable tycho build extension -->
				<groupId>org.eclipse.tycho</groupId>
				<artifactId>tycho-maven-plugin</artifactId>
				<version>${tycho-version}</version>
				<extensions>true</extensions>
			</plugin>
			<plugin>
				<groupId>org.eclipse.tycho</groupId>
				<artifactId>target-platform-configuration</artifactId>
				<version>${tycho-version}</version>
				<configuration>
					<pomDependencies>consider</pomDependencies>
					<!-- configure the p2 target environments for multi-platform build -->
					<environments>
						<environment>
							<os>linux</os>
							<ws>gtk</ws>
							<arch>x86_64</arch>
						</environment>
						<environment>
							<os>win32</os>
							<ws>win32</ws>
							<arch>x86_64</arch>
						</environment>
					</environments>
				</configuration>
			</plugin>
			<!-- enable source bundle generation -->
			<plugin>
				<groupId>org.eclipse.tycho</groupId>
				<artifactId>tycho-source-plugin</artifactId>
				<version>${tycho-version}</version>
				<executions>
					<execution>
						<id>plugin-source</id>
						<goals>
							<goal>plugin-source</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
	<modules>

		<module>../org.nasdanika.tools.feature</module>
		<module>../org.nasdanika.tools.repository</module>

	</modules>
	<name>Nasdanika Tool Suite</name>
	<url>https://www.nasdanika.org/products/tools</url>
	<organization>
		<name>Nasdanika LLC</name>
		<url>http://www.nasdanika.org</url>
	</organization>
	<scm>
		<url>https://github.com/Nasdanika/nasdanika-tools-suite</url>
	</scm>
</project>
```

## Feature

The feature project contains ``feature.xml`` and ``pom.xml``. It may also contain a target definition file to simplify editing of the feature.

### feature.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<feature
      id="org.nasdanika.tools.feature"
      label="Nasdanika Tool Suite"
      version="0.1.0.qualifier">

   <description>
      Nasdanika and third-party tools.
   </description>

   <copyright>
      Property of Nasdanika LLC, all rights reserved.
   </copyright>

   <license url="https://www.eclipse.org/legal/epl-v10.html">
      Eclipse public license
   </license>

   <url>
      <discovery label="Eclipse Photon" url="http://download.eclipse.org/releases/photon"/>
      <discovery label="Tycho Connector" url="http://repo1.maven.org/maven2/.m2e/connectors/m2eclipse-tycho/0.8.1/N/0.8.1.201704211436/"/>
      <discovery label="Nasdanika" url="https://www.nasdanika.org/products/site/repository"/>
   </url>

   <includes
         id="org.eclipse.m2e.feature"
         version="0.0.0"/>

   <includes
         id="org.sonatype.tycho.m2e.feature"
         version="0.0.0"/>
         
   <includes
         id="org.eclipse.wst.web_ui.feature"
         version="0.0.0"/>
             
   <!-- <includes id="org.nasdanika.config.feature" version="0.0.0"/> - Included in codegen -->
         
   <!-- <includes id="org.nasdanika.codegen.feature" version="0.0.0"/> - Included in codegen.ecore -->

   <includes id="org.nasdanika.codegen.ecore.feature" version="0.0.0"/>
         
   <includes id="org.nasdanika.codegen.ecore.web.ui.feature" version="0.0.0"/>

   <includes id="org.nasdanika.docgen.ecore.feature" version="0.0.0"/>
         
   <includes id="org.nasdanika.workspace.wizard.feature" version="0.0.0"/>

</feature>
```

### pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <artifactId>org.nasdanika.tools.feature</artifactId>
  <packaging>eclipse-feature</packaging>
  <name>Nasdanika Tool Suite Feature</name>
  <parent>
  	<groupId>org.nasdanika.tools</groupId>
  	<artifactId>org.nasdanika.tools.parent</artifactId>
  	<version>0.1.0-SNAPSHOT</version>
  	<relativePath>../org.nasdanika.tools.parent</relativePath>
  </parent>
</project>
```

## Repository

The repository project is responsible for building:
* P2 repository.
* Site.
* Binary distribution.

### pom.xml

This file is used to build the site and the P2 repository as part of the parent build.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<artifactId>org.nasdanika.tools.repository</artifactId>
	<name>Nasdanika Tool Suite Repository</name>
	<packaging>eclipse-repository</packaging>
	<parent>
		<groupId>org.nasdanika.tools</groupId>
		<artifactId>org.nasdanika.tools.parent</artifactId>
		<version>0.1.0-SNAPSHOT</version>
		<relativePath>../org.nasdanika.tools.parent</relativePath>
	</parent>
	<build>
		<extensions>
			<!-- Enabling the use of FTP -->
			<extension>
				<groupId>org.apache.maven.wagon</groupId>
				<artifactId>wagon-ftp</artifactId>
				<version>3.0.0</version>
			</extension>
		</extensions>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
			    <artifactId>maven-site-plugin</artifactId>
			    <version>3.7.1</version>
			    <configuration>
			        <skip>false</skip>
			        <skipDeploy>false</skipDeploy>
			    </configuration>
			</plugin>		
			<plugin>
				<artifactId>maven-resources-plugin</artifactId>
				<version>3.0.1</version>
				<executions>
					<execution>
						<id>copy-resources</id>
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
										<include>org.nasdanika.tools.repository-*.zip</include>
									</includes>
									<filtering>false</filtering>
								</resource>
							</resources>
						</configuration>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.eclipse.tycho</groupId>
				<artifactId>tycho-p2-repository-plugin</artifactId>
				<version>${tycho-version}</version>
				<configuration>
					<includeAllDependencies>true</includeAllDependencies>
				</configuration>
			</plugin>
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
								<ftp passive="yes" action="delete" server="${env.FTP_SERVER}" userid="${env.FTP_USER}" password="${env.FTP_PASSWORD}" remotedir="/tools">
									<fileset defaultexcludes="false">
										<include name="**" />
									</fileset>
								</ftp>
								
								<ftp passive="yes" action="rmdir" server="${env.FTP_SERVER}" userid="${env.FTP_USER}" password="${env.FTP_PASSWORD}" remotedir="/tools">
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
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<version>3.1.0</version>
				<configuration>
					<skipAssembly>false</skipAssembly>
					<outputDirectory>${project.build.directory}/site</outputDirectory>
					<formats>zip</formats>
					<finalName>nasdanika-tool-suite-source</finalName>
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
		</plugins>
	</build>
	<distributionManagement>
		<site>
			<id>nasdanika-org</id>
			<url>ftp://${env.FTP_SERVER}/tools</url>
		</site>
	</distributionManagement>	
	
</project>
``` 

### pom-binary.xml

This file is used to build a package binary distribution. This build is invoked individually and shall be performed after the P2 repository is deployed.

In the snippet below the base package is Eclipse Modeling Package, replace the URL as appropriate for your needs.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<artifactId>org.nasdanika.tools.binary</artifactId>
	<groupId>org.nasdanika.tools</groupId>
	<version>0.1.0-SNAPSHOT</version>
		
	<name>Nasdanika Tool Suite Binary Distribution</name>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-antrun-plugin</artifactId>
				<version>1.8</version>
				<executions>
					<execution>
						<id>ftp</id>
						<phase>generate-resources</phase>
						<configuration>
							<target>
								<!-- Download binary -->
								<get src="http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/photon/R/eclipse-modeling-photon-R-win32-x86_64.zip&amp;r=1" dest="${project.build.directory}/eclipse.zip"/>
								
								<!-- Unzip -->
								<unzip src="${project.build.directory}/eclipse.zip" dest="${project.build.directory}"/>
								
								<!-- Call P2 Director -->
								<java 
									jar="${project.build.directory}/eclipse/plugins/org.eclipse.equinox.launcher_1.5.0.v20180512-1130.jar" 
									dir="${project.build.directory}/eclipse"
									fork="true" 
									failonerror="true"
									maxmemory="512m">

									<arg line="-application org.eclipse.equinox.p2.director -consoleLog -nosplash -repository https://www.nasdanika.org/products/tools/repository -installIUs org.nasdanika.tools.feature.feature.group -tag NasdanikaToolSuite" />
								</java>
								
								<!-- Zip -->
								<zip 
									destfile="${project.build.directory}/nasdanika-tool-suite-0.1.0-eclipse-modeling-photon-R-win32-x86_64.zip" 
									basedir="${project.build.directory}"
									includes="eclipse/**"/>
								 
								<!-- Upload binary -->
								<ftp 
									passive="yes" 
									server="${env.FTP_SERVER}" 
									userid="${env.FTP_USER}" 
									password="${env.FTP_PASSWORD}" 
									remotedir="/tools">
									
									<fileset 
										dir="${project.build.directory}" 
										includes="nasdanika-tool-suite-0.1.0-eclipse-modeling-photon-R-win32-x86_64.zip"/>
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
		</plugins>
	</build>	
</project>
```

## Running

* To build and deploy the repository and site invoke ``mvn clean package site-deploy`` from the parent project.
* To build and deploy the binary distribution invoke ``mvn -f pom-binary.xml generate-resources`` from the repository project.

## CircleCI 

If you are planning to build with [CircleCI](https://circleci.com/) then create ``.circleci`` directory in the root of the source repository - at the same level as the projects with the following files:

### config.yml

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
      - run: cd org.nasdanika.tools.parent; mvn -s ../.circleci/settings.xml clean package site-deploy
      - run: cd org.nasdanika.tools.repository; mvn -f pom-binary.xml -s ../.circleci/settings.xml generate-resources
```

### settings.xml

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

You will need to define the following environment variables:
* FTP_SERVER
* FTP_USER
* FTP_PASSWORD

