# fuse - parent profile

# Overview
This example showcases the use of the Fabric8 (v1) profile hierarchy to deal with environment-dependent properties.
An abstract parent profile is used to gather all the environment-specific properties in one or many files.
Each application defines this profile as its (abstract) parent profile and can therefore inherit the environment-specific properties as PID properties from the profile hierarchy.

The parent project is built against a Maven "build profile" that corresponds to one environment.
The Maven resource plugin is customized in each "build profile" to load the properties corresponding to the right environment.

# Compatibility
The below example is built for Fuse v6.2 in Fabric mode.

# Detailed description

The Maven project is made of 3 parts: a parent module and 2 submodules.

## fabricenv
  The parent module gathers the information reusable by the submodules like:
  - The bom file of Fuse v6.2
  - The declaration of the maven bundle plugin 
    We use fabric8-maven-plugin v1.1.0.CR5 instead of v1.2.0.redhat-133
    This version of the plugin requires an extra library available @ https://repository.jboss.org/
    Notice that the maven deploy goal changes the "version" attribute to a timestamped version when working with snapshots.
    If combined with the "fabric8:deploy" goal, this leads to a fabric8 profile that contains wrong maven coordinates.  

## environment
  The "fabric_env" submodule is the environment-dependent profile.
  It only contains properties; no binary code.
  Within fabric, it will be the parent profile of all the Fabric8 applications.
  In the example it uses an env.properties file declared under src/main/fabric8
  The module is built against a specific maven profile (here dev or prd).
  We choose to store a different env.properties file under a directory corresponding to an environment (dev & prd).
  The profile will copy the corresponding file into /src/main/fabric8 using the maven resources plugin and the copy-file action.
  Another possibility would for example be to change the "profileConfigDir" parameter of the fabric8-maven-plugin to directly read the properties from the "dev" or "prd" according to the chosen profile. 
  
  The module also provides specific values for some fabric8-maven-plugin properties (here it simply declares the profile as abstract).

## app
  The "fabric_app" submodule simulates a Fabric8 application.
  It declares a blueprint property placeholder with a pid value of "env", which corresponds to the "env.properties" file exposed by the parent profile.
  It also declares the Fabric8 "fabric_env" profile as its parent.
  The Blueprint property placeholder element belongs to the "http://aries.apache.org/blueprint/xmlns/blueprint-cm/v1.0.0" namespace.
  The elements from this namespace are provided by the camel-blueprint feature which is declared as a dependency feature through a fabric8 property.
 
  As for the code, the application simply logs the values of some of the environment-specific variables every 5 sec

# Demo
For a better experience with the example, install Fuse v6 and create 2 Fabrics; one for dev and one for prod.

 - build the fabric_env profile against the "dev" Maven profile and deploy it to the "dev" environment.
   "mvn -U package fabric8:deploy -Pdev -DjolokiaUrl=http://<dev_address>/jolokia"
 - build the fabric_env profile with the "prd" Maven profile and deploy it to the "prod" environment.
   "mvn -U package fabric8:deploy -Pprd -DjolokiaUrl=http://<prod_address>/jolokia"

 - deploy the exact same fabric_app profile twice, one in each environemnt
   "mvn -U package fabric8:deploy -DjolokiaUrl=http://<dev_address>/jolokia"
   "mvn -U package fabric8:deploy -DjolokiaUrl=http://<prod_address>/jolokia"

 - create a "default" child container in each Fabric and assign the fabric_app profile to it
 - check the log.  
   you should see the application logging some environment-specific properties values every 5 sec.
   though it's the exact same binary code in both environment, you'll see different environment values

# Versioning the environment-specific profile
The fabric_env profile contains environment variables that can be seen in the Fuse Management Console.
However it's not expected that the values be updated from the Web Console.
The code should be stored in an SCM and the value of the environment properties should be updated in a release circle starting from the SCM repository.

The applications must not change when the value of some environment properties changes.
Therefore the fabric_env profile must not contain any version attribute in its name.
We can use the Fabric8 versioning mechanism to release such a component:
 - Create a new version of the Fabric (fabric-version-create)
   Notice that the version number can correspond to a date instead of a standart application version
 - Deploy a new fabric_env profile to that version
   Virtually it's like you had 2 Fabrics, one for the "old" environment and one for the "new" one
 - Migrate the containers to that new environment
 
# Fuse without Fabric
The principle is applicable with Fuse standalone.
In that case, the environment-dependent property file(s) will simply have to be placed in $FUSE_HOME/etc (with a .cfg extension instead of .properties).


# Links
* [Fabric8 maven plugin](http://fabric8.io/gitbook/mavenPlugin.html)

