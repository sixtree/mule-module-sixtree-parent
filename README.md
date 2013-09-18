Sixtree Mule Modules Parent POM
===============================

Parent project for all Sixtree Mule Module development, primarily for Devkit development.

Features
========

The Sixtree Mule Modules Parent POM features the following:

* Based off the MuleSoft Devkit Parent POM, with relevant overrides and defaults
* Configured to release binaries to the Sixtree S3 based public Maven repo
* Configured to use the Sixtree Github source repo
* Capable of preparing and signing the Eclipse update site for Mule Studio plugins (if applicable)

The following are currently missing:

* Automatic upload of Mule Studio plugins to the Sixtree p2 repository
* A Maven Archetype that begins a project with all the relevant changes from the Mule Devkit Archetype (currently performed manually)

Usage
=====

This POM is intended to support the full development and release lifecycle of public Sixtree Mule Modules.

## Configure

The following steps are necessary for using the POM to build and release:

* Install Maven and have the `mvn` command available in your path
* Edit your `M2_HOME/conf/settings.xml` to include the following:

```xml
<servers>
  ...
  <server>
    <id>sixtree-releases</id>
    <username>AAAAAAAAAAAAAAAAAAAA</username>                      <!-- AWS Access Key -->
    <password>BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB</password>  <!-- AWS Secret     -->
  </server>
  <server>
    <id>sixtree-snapshots</id>
    <username>AAAAAAAAAAAAAAAAAAAA</username>                      <!-- AWS Access Key -->
    <password>BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB</password>  <!-- AWS Secret     -->
  </server>
  ...
</servers>
```

#### Optional Configuration

If building the module directly from Mule Studio (as opposed to the using `mvn` on the command line), also:

* Edit your `M2_HOME/conf/settings.xml` to include the following:

```xml
<pluginGroups>
  ...
  <pluginGroup>org.mule.tools</pluginGroup>
  ...
</pluginGroups>
```

* Ensure your Mule Studio is being launched using a JDK and *not* a JRE by editing `MuleStudio.ini` in the Mule Studio installation directory, and adding the following lines **before** the `-vmargs` line:

```
-vm
<path to JDK home>/bin/javaw.exe
```

**Note**: It is important to ensure these entries are on separate lines. Any issues getting Mule Studio to start after these changes should first be debugged by checking here: http://wiki.eclipse.org/Eclipse.ini

## Start a Module

The following example assumes that the module being built is called *Example*. Usually it is simply the name of the technology being interfaces with, or some grouping of functionality (such as 'xref', 'logging' or 'common').

#### Github

First, set up a Github repo for this module, *but **don't** initialise it*:

* *Github Repo* - http://github.com/sixtree/mule-module-example

Next, initialise and push to this repo, while modifying the main branch: 

```
mkdir mule-module-example && cd mule-module-example
git init
touch README.md
git add README.md
git commit -m "first commit"
git branch -m 1.x
git remote add origin https://github.com/sixtree/mule-module-example.git
git push -u origin 1.x
```

Note that if using an existing project, modify the `mkdir` and `git add` lines as appropriate.

#### Maven

Use Maven to create the project archetype in the initialised directory by following the instructions at [Building a Connector with Maven and Studio](http://www.mulesoft.org/documentation/display/current/Building+a+Connector+With+Maven+and+Studio). However, you **must** use the following conventions instead:

* *groupId* - au.com.sixtree.mule
* *artifactId* - mule-module-example
* *muleConnectorName* - Example
* *package* - au.com.sixtree.mule.modules.example
* *version* - 1.0.0-SNAPSHOT (initially)

Once the project is created, modify the resulting modules `pom.xml` as follows.

* Remove the `groupId`
* Add the following parent POM reference:

```xml
<parent>
	<groupId>au.com.sixtree.mule</groupId>
	<artifactId>mule-module-sixtree-parent</artifactId>
	<version>RELEASE</version>
</parent>
```

* Ensure the properties are set appropriately (override parent settings where necessary).
* Add the url:

```xml
<url>http://github.com/sixtree/mule-module-example</url>
```

* Add the Github details:

```xml
<scm>
	<connection>scm:git:git://github.com:sixtree/mule-module-example.git</connection>
	<developerConnection>scm:git:git@github.com:sixtree/mule-module-example.git</developerConnection>
	<url>http://github.com/sixtree/mule-module-example</url>
</scm>
```

* Add the Sixtree repositories (so this POM can be found):

```xml
<repositories>
  ...
	<repository>
		<id>sixtree-releases</id>
		<name>Sixtree Repository</name>
		<url>http://dist.sixtree.com.au/releases/</url>
	</repository>
	<repository>
		<id>sixtree-snapshots</id>
		<name>Sixtree Snapshot Repository</name>
		<url>http://dist.sixtree.com.au/snapshots/</url>
	</repository>
	...
</repositories>
```

**Note**: There may be settings in this parent `pom.xml` that you don't wish to have in the actual module (such as, for example, Studio packaging). Modify the modules `pom.xml` to override these defaults as appropriate.

#### Miscellaneous

The default Sixtree LICENSE.md file can be found here: https://github.com/sixtree/mule-module-sixtree-parent/blob/1.x/LICENSE.md

## Build

The build process simply leverages Maven. 

To build and install locally (to use in a Mule project or to import into Mule Studio as a plugin):

```
mvn install
```

To deploy snapshot releases to the Sixtree repo as well (**avoid doing this unless absolutely necessary**):

```
mvn install deploy
```

## Eclipse Plugin

By default, the Maven build creates a `target\update-site` directory. This is unsigned and will generate a warning in Mule Studio on import.

To sign the `update-site` on builds, add the following to your `M2_HOME/conf/settings.xml` file:

```xml
<profiles>
  ...
  <profile>
      <id>sign</id> 
      <activation>
          <activeByDefault>false</activeByDefault> 
      </activation>
      <properties>
         <alias>STORE_ALIAS</alias>
         <keystore.path>/path/to/keystore.ks</keystore.path>
         <storepass>STORE_PASSWORD</storepass>
         <keypass>KEY_PASSWORD</keypass>
      </properties>
  </profile>
  ...
</profiles>
```

And activate the profile when running the `install` or `release:prepare` commands:

```
mvn -Psign install
```

## Release

**Note**: The following relies on the correct configuration of your git installation to have Github credentials for HTTPS access already configured. This differs between operating systems and is not part of this guide.

Building and releasing to the Sixtree repo is a simple process also:

```
mvn -Psign release:prepare
mvn -Psign release:perform
```

Note sometimes its useful to skip tests on the perform step (if the tests require sensitive information that is not checked in). Just run as follows:

```
mvn -Psign -Darguments="-Dmaven.test.skip=true" release:perform
```

Usually the defaults for release version and next development version are acceptable for fixes (patches). However, remember to always follow the [Semantic Versioning](http://semver.org/) conventions and increment MAJOR and MINOR if appropriate. If incrementing the MAJOR version (breaking changes), a new root branch is required on Github called <MAJOR>.x (for example, 2.x).

The above results in a newly released Maven artefact in http://dist.sixtree.com.au/releases. However, **the Eclipse update site is not installed**. To install the update site manually, either:

* Get the update-site folder from the target/checkout folder immediately after release, or
* Download the update site zip from the distribution site release folder (example URL is https://s3-ap-southeast-2.amazonaws.com/dist.sixtree.com.au/releases/au/com/sixtree/mule/mule-module-example/1.0.0/mule-module-example-1.0.0-studio-plugin.zip)

The target location to copy the entire update-site folder is s3://dist.sixtree.com.au/mule/modules/mule-module-example/update-site

## Documentation

The default `mvn install` command incorrectly generates documentation for Mule Modules (a known bug, reference can't be found at the moment). To generate the correct documentation, run the `javadoc` target directly:

```
mvn javadoc:javadoc
```

This can then be released to Github Pages using the following:

```
mvn -Dgithub.oauth2Token=<token> mule-devkit:github-upload-doc
```

The resulting documentation can be found at http://sixtree.github.io/mule-module-example

#### Extra Documentation

Any markdown flavoured guides can be added to the default documentation by replacing the default `maven-javadoc-plugin` configuration in the parent POM. Specifically, the following lines:

```xml
<plugin>
	<artifactId>maven-javadoc-plugin</artifactId>
	<version>2.8</version>
	<configuration>
		<excludePackageNames>org.mule.tooling.ui.contribution:*.xsd:*</excludePackageNames>
		<additionalparam>
			-hdf project.artifactId "${project.artifactId}"
			-hdf project.groupId "${project.groupId}"
			-hdf project.version "${project.version}"
			-hdf project.name "${project.name}"
			<!-- release repos are hardcoded because distribution management contains 
				s3:// urls -->
			-hdf project.repo.name "Sixtree Repository"
			-hdf project.repo.id "sixtree-releases"
			-hdf project.repo.url "http://dist.sixtree.com.au/releases/"
			-hdf project.snapshotRepo.name "Sixtree Snapshot Repository"
			-hdf project.snapshotRepo.id "sixtree-snapshots"
			-hdf project.snapshotRepo.url "http://dist.sixtree.com.au/snapshots/"
			-d ${project.build.directory}/apidocs
			<!-- -markdown userguide ${basedir}/USAGE.md "User Guide" -->
		</additionalparam>
	</configuration>
</plugin>
```

The commented section can be used to add other pages not usually part of the default Mule documentation:

```
-markdown userguide ${basedir}/USAGE.md "User Guide"
```


Changelog
=========

#### 1.0.0 - 2013-09-08

- Initial release
