# Single Site Eclipse Workspace

This guide explains how to build a "workspace" - a set of related projects - using [Tycho](https://www.eclipse.org/tycho/)/Maven and featuring a single site with a [P2](https://www.eclipse.org/equinox/p2/) repository and product binaries for [Eclipse products](https://wiki.eclipse.org/FAQ_What_is_an_Eclipse_product%3F]. 

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

It tells Maven to skip site building and deploying for all plug-ins unless overriden.