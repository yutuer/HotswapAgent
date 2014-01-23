Hotswap Agent
=============
Java unlimited runtime class and resource redefinition.

The main purpose of this project is to avoid infamous change->restart + *wait*->check development lifecycle.
Save&Reload during development should be standard and many other languages (including C#) contain this feature.

This project is still in a beta version.

### Easy to start
Download and install DCEVM Java patch + agent jar and launch your application server with options
`-XXaltjvm=dcevm -javaagent:HotswapAgent.jar` to get working setup. Optionally add hotswap-agent.properties
to your application to configure plugins and agent behaviour.

### Plugins
Each application framework (Spring, Hibernate, Logback, ...) needs special reloading mechanism to keep
up-to-date after class redefinition (e.g. Hibernate configuration reload after new entity class is introduced).
Hotswap agent works as a plugin system and ships preconfigured with all major framework plugins. It is easy
to write your custom plugin even as part of your application.

### IDE support
None needed :) Really, all changes are transparent and all you need to do is to download patch+agent and
setup your application / application server. Because we use standard java hotswap behaviour, your IDE will
work as expected. However, we work on IDE plugins to help with download & configuration.

Quick start:
===========
### Install
1. download [latest release](https://github.com/HotswapProjects/HotswapAgent/releases/download/0.1-beta1/HotswapAgent-0.1-beta1.zip)
 and unpack it's contents. You will need `jvm.dll` and `HotswapAgent.jar` files.
1. check that you have installed *JDK 1.7.0_45 Windows 64bit*, otherwise download and install [from here]
(http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)
1. install patched Java Hotspot(r) file: inside the JDK 1.7.0_45 installation directory create new directory "jre\bin\dcevm"
and put `jvm.dll` into it. For example: `C:\Program Files\Java\jdk1.7.0_45\jre\bin\dcevm\jvm.dll`. (Should you need
other platform/version, currently you need to compile the file yourself from the [source](https://github.com/Guidewire/DCEVM).
1. unpack `HotswapAgent.jar` and put it anywhere on your disc. For example: `C:\java\HotswapAgent.jar`

### Run your application
1. use your favorite IDE of choice
1. add following command line java attributes:
  <pre>-XXaltjvm=dcevm -javaagent:PATH_TO_AGENT\HotswapAgent.jar</pre> You need to replace PATH_TO_AGENT with an actual
  directory. For example `java -XXaltjvm=dcevm -javaagent:c:\java\HotswapAgent.jar YourApp`.
  See [IntelliJ IDEA](https://groups.google.com/forum/#!topic/hotswapagent/BxAK_Clniss)
  and [Netbeans](https://groups.google.com/forum/#!topic/hotswapagent/ydW5bQMwQqU) forum threads for IDE specific setup guides.
1. (optional) create a file named "hotswap-agent.properties" inside your resources direcotry, see available properties and
  default values: <https://github.com/HotswapProjects/HotswapAgent/blob/master/HotswapAgent/src/main/resources/hotswap-agent.properties>
1. start the application in debug mode.
1. save a resource and/or use the HotSwap feature of your IDE to reload changes

### What is available?
* Enhanced Java Hotswap - change method body, add/rename a method, field, ... The only unsupported operation
  is hierarchy change (change the superclass or remove an interface).
* Reload resource - resources from webapp directory are usually reloaded by application server. But what about
  other resources like src/main/resources? Use watchResources property to add any directory to watch for a resource change.
* Extra classpath - Need change of a class inside dependent jar? Use extraClasspath property to add any directory as
  a classpath to watch for class files
* Framework support - through plugin system, many fra
* Reload without IDE - you can configure the agent to automatically reload changed class file automatically (without IDE).
  This may be used to upload changed classes even on a production system without restart (note, that the agent is not stable
  enough yet, use at your own risk).
* Fast - until the plugin is initialized, it does not consume any resources or slow down the application (see Runtime overhead for more information)

Should you have any problems or questions, ask at [HotswapAgent forum](https://groups.google.com/forum/#!forum/hotswapagent).

Configuration
=============
The basic configuration is configured reload classes and resources from classpath known to the running application
(classloader). If you need a different configuration, add hotswap-agent.properties file to the classpath root
(e.g. `src/main/resources/hotswap-agent.properties`).

Detail documentation of available properties and default values can be found in the [agent properties file](https://github.com/HotswapProjects/HotswapAgent/HotswapAgent/blob/master/src/main/resources/hotswap-agent.properties)


How does it work?
=================

### DCEVM
Hotswap agent does the work of reloading resources and framework configuration (Spring, Hibernate, ...),
but it depends on standard Java hotswap mechanism to actually reload classes. Standard Java hotswap allows
only method body change , which makes it practically unusable. DCEVM is a JRE patch witch allows almost any
structural class change on hotswap (with an exception of a hierarchy change). Although hotswap agent works
even with standard java, we recommend to use DCEVM (and all tutorials use DCEVM as target JVM).

### Hotswap agent
Hotswap agent is a plugin container with plugin manager, plugin registry, and several agent services
(e.g. to watch for class/resource change). It helps with common tasks and classloading issues. It scans classpath
for class annotated with @Plugin annotation, injects agent services and registers reloading hooks. Runtime bytecode
modification is provided by javaasist library.

### Plugins
Plugins administered by Hotswap agent are usually targeted towards a specific framework. For example Spring plugin
uses agent services to:
* Modify root Spring classes to get Spring contexts and registered scan path
* Watch for any resource change on a scan path
* Watch for a hotswap of a class file within a scan path package
* Reload bean definition after a change
* ... and many other

Packaged plugins:
* Hibernate (4x) - Reload Hibernate configuration after entity create/change.
* Spring (3x) - Reload Spring configuration after class definition/change.
* Jetty - add extra classpath to the app classloader. All versions supporting WebAppContext.getExtraClasspath should be supported.
* ZK (5x-7x) - ZK Framework (http://www.zkoss.org/). Change library properties default values to disable caches, maintains Label cache and
bean resolver cache.
* Logback - Logback configuration reload
* Hotswapper - Watch for any class file change and reload (hotswap) it on the fly via Java Platform Debugger Architecture (JPDA)
* AnonymousClassPatch - Swap anonymous inner class names to avoid not compatible changes.

Find a detail documentation of each plugin in the plugin project main README.md file.

### Runtime overhead
It really depends on how many frameworks you use and which caches are disabled. Example measurements
for a large, real world enterprise application based on Spring + Hibernate, run on Jetty.

    Setup                        | Startup time
    -----------------------------|-------------
    Run (plain Java)             | 23s
    Debug (plain Java)           | 24s
    Debug (plain DCEVM)          | 28s
    Agent - disabled all plugins | 31s
    Agent - all plugins          | 35s


How to write a plugin
=====================
(check exiting plugins)
TODO documentation

Credits
=======
Hotswap agent:
* Jiri Bubnik - project coordinator, initial implementation
* Jan Tecl - web design (work in progress)

DCEVM:
* Thomas Würthinger - project coordinator, initial implementation.
* Kerstin Breitender – contributor.
* Christoph Wimberger – contributor.
* Ivana Dubrov - update to Java7, patches, build system (Gradle)

