

# Preface

Welcome to the Hazelcast IMDG (In-Memory Data Grid) Reference Manual. This manual includes concepts, instructions, and samples to guide you on how to use Hazelcast and build Hazelcast IMDG applications.

As the reader of this manual, you must be familiar with the Java programming language and you should have installed your preferred Integrated Development Environment (IDE).

## Hazelcast IMDG Editions

This Reference Manual covers all editions of Hazelcast IMDG. Throughout this manual:

- **Hazelcast** or **Hazelcast IMDG** refers to the open source edition of Hazelcast in-memory data grid middleware. **Hazelcast** is also the name of the company (Hazelcast, Inc.) providing the Hazelcast product.
- <font color="#3981DB">**Hazelcast IMDG Enterprise**</font> is a commercially licensed edition of Hazelcast IMDG which provides high-value enterprise features in addition to Hazelcast IMDG.
- <font color="##153F75">**Hazelcast IMDG Enterprise HD**</font> is a commercially licensed edition of Hazelcast IMDG which provides High-Density (HD) Memory Store and Hot Restart Persistence features in addition to Hazelcast IMDG Enterprise.


## Hazelcast IMDG Architecture

You can see the features for all Hazelcast IMDG editions in the following architecture diagram.

![Hazelcast Architecture](images/HazelcastArchitecture.png)

![image](images/NoteSmall.jpg) ***NOTE*** *You can see small "HD" boxes for some features in the above diagram. Those features can use High-Density (HD) Memory Store when it is available. It means if you have Hazelcast IMDG Enterprise HD, you can use those features with HD Memory Store.*

For more information on Hazelcast IMDG's Architecture, please see the white paper <a href="https://hazelcast.com/resources/architects-view-hazelcast/" target="_blank">An Architect's View of Hazelcast</a>.

## Hazelcast IMDG Plugins

You can extend Hazelcast IMDG's functionality by using its plugins. These plugins have their own lifecycles. Please see <a href="https://hazelcast.org/plugins/" target="_blank">Plugins page</a> to learn about Hazelcast plugins you can use. Hazelcast plugins are marked with <img src="images/Plugin_New.png" alt="Plugin" height="12" width="54"> label throughout this manual.


## Licensing

Hazelcast IMDG and Hazelcast Reference Manual are free and provided under the Apache License, Version 2.0. Hazelcast IMDG Enterprise and Hazelcast IMDG Enterprise HD is commercially licensed by Hazelcast, Inc.

For more detailed information on licensing, please see the [License Questions appendix](#license-questions).


## Trademarks

Hazelcast is a registered trademark of Hazelcast, Inc. All other trademarks in this manual are held by their respective owners. 


## Customer Support

Support for Hazelcast is provided via <a href="https://github.com/hazelcast/hazelcast/issues" target="_blank">GitHub</a>, <a href="https://groups.google.com/forum/#!forum/hazelcast" target="_blank">Mail Group</a> and <a href="http://www.stackoverflow.com" target="_blank">StackOverflow</a>

For information on the commercial support for Hazelcast IMDG and Hazelcast IMDG Enterprise, please see 
<a href="https://hazelcast.com/pricing/" target="_blank">hazelcast.com</a>.

## Release Notes

Please refer to the <a href="http://docs.hazelcast.org/docs/release-notes/" target="_blank">Release Notes document</a> for the new features, enhancements and fixes performed for each Hazelcast IMDG release.



## Contributing to Hazelcast IMDG

You can contribute to the Hazelcast IMDG code, report a bug, or request an enhancement. Please see the following resources.

- <a href="https://hazelcast.atlassian.net/wiki/display/COM/Developing+with+Git" target="_blank">Developing with Git</a>: Document that explains the branch mechanism of Hazelcast and how to request changes.
- <a href="https://hazelcast.atlassian.net/wiki/display/COM/Hazelcast+Contributor+Agreement" target="_blank">Hazelcast Contributor Agreement form</a>: Form that each contributing developer needs to fill and send back to Hazelcast.
- <a href="https://github.com/hazelcast/hazelcast" target="_blank">Hazelcast on GitHub</a>: Hazelcast repository where the code is developed, issues and pull requests are managed.

## Partners

Hazelcast partners with leading hardware and software technologies, system integrators, resellers and OEMs including Amazon Web Services, Vert.x, Azul Systems, C2B2. Please see the [Partners](https://hazelcast.com/partners/) page for the full list of and information on our partners.


## Phone Home

Hazelcast uses phone home data to learn about usage of Hazelcast IMDG.

Hazelcast IMDG member instances call our phone home server initially when they are started and then every 24 hours. This applies to all the instances joined to the cluster.

**What is sent in?**

The following information is sent in a phone home:

- Hazelcast IMDG version
- Local Hazelcast IMDG member UUID
- Download ID 
- A hash value of the cluster ID
- Cluster size bands for 5, 10, 20, 40, 60, 100, 150, 300, 600 and > 600
- Number of connected clients bands of 5, 10, 20, 40, 60, 100, 150, 300, 600 and > 600
- Cluster uptime
- Member uptime
- Environment Information:
	- Name of operating system
	- Kernel architecture (32-bit or 64-bit)
	- Version of operating system
	- Version of installed Java
	- Name of Java Virtual Machine
- Hazelcast IMDG Enterprise specific: 
	- Number of clients by language (Java, C++, C#)
	- Flag for Hazelcast Enterprise 
	- Hash value of license key
	- Native memory usage
- Hazelcast Management Center specific: 
    - Hazelcast Management Center version
    - Hash value of Hazelcast Management Center license key

**Phone Home Code**

The phone home code itself is open source. Please see <a href="https://github.com/hazelcast/hazelcast/blob/master/hazelcast/src/main/java/com/hazelcast/util/PhoneHome.java" target="_blank">here</a>.

**Disabling Phone Homes**

Set the `hazelcast.phone.home.enabled` system property to false either in the config or on the Java command line. Please see the [System Properties section](#system-properties) for information on how to set a property. 

Starting with Hazelcast 3.9, you can also disable the phone home using the environment variable `HZ_PHONE_HOME_ENABLED`. Simply add the following line to your `.bash_profile`:

```
export HZ_PHONE_HOME_ENABLED=false
```

**Phone Home URLs**

For versions 1.x and 2.x: <a href="http://www.hazelcast.com/version.jsp" target="_blank">http://www.hazelcast.com/version.jsp</a>.

For versions 3.x up to 3.6: <a href="http://versioncheck.hazelcast.com/version.jsp" target="_blank">http://versioncheck.hazelcast.com/version.jsp</a>.

For versions after 3.6: <a href="http://phonehome.hazelcast.com/ping" target="_blank">http://phonehome.hazelcast.com/ping</a>.



## Typographical Conventions

Below table shows the conventions used in this manual.

|Convention|Description|
|:-|:-|
|**bold font**| - Indicates part of a sentence that requires the reader's specific attention. <br> - Also indicates property/parameter values.|
|*italic font*|- When italicized words are enclosed with "<" and ">", it indicates a variable in the command or code syntax that you must replace (for example, `hazelcast-<`*version*`>.jar`). <br> - Note and Related Information texts are in italics.|
|`monospace`|Indicates files, folders, class and library names, code snippets, and inline code words in a sentence.|
|***RELATED INFORMATION***|Indicates a resource that is relevant to the topic, usually with a link or cross-reference.|
|![image](images/NoteSmall.jpg) ***NOTE***| Indicates information that is of special interest or importance, for example an additional action required only in certain circumstances.|
|element & attribute|Mostly used in the context of declarative configuration that you perform using Hazelcast XML file. Element refers to an XML tag used to configure a Hazelcast feature. Attribute is a parameter owned by an element, contributing into the declaration of that element's configuration. Please see the following example.<br></br>`<port port-count="100">5701</port>`<br></br> In this example, `port-count` is an **attribute** of the `port` **element**.


<br></br>







