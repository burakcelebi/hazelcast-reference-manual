
Hazelcast IMDG uses phone home data to learn about its usage.

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
	- Flag for Hazelcast IMDG Enterprise 
	- Hash value of license key
	- Native memory usage
- Hazelcast Management Center specific: 
    - Hazelcast Management Center version
    - Hash value of Hazelcast Management Center license key

**Phone Home Code**

The phone home code itself is open source. Please see <a href="https://github.com/hazelcast/hazelcast/blob/master/hazelcast/src/main/java/com/hazelcast/util/PhoneHome.java" target="_blank">here</a>.

**Disabling Phone Homes**

Set the `hazelcast.phone.home.enabled` system property to false either in the config or on the Java command line. Please see the [System Properties section](../25_System_Properties.md) for information on how to set a property.

Starting with Hazelcast 3.9, you can also disable the phone home using the environment variable `HZ_PHONE_HOME_ENABLED`. Simply add the following line to your `.bash_profile`:

```
export HZ_PHONE_HOME_ENABLED=false
```

**Phone Home URLs**

For versions 1.x and 2.x: <a href="http://www.hazelcast.com/version.jsp" target="_blank">http://www.hazelcast.com/version.jsp</a>.

For versions 3.x up to 3.6: <a href="http://versioncheck.hazelcast.com/version.jsp" target="_blank">http://versioncheck.hazelcast.com/version.jsp</a>.

For versions after 3.6: <a href="http://phonehome.hazelcast.com/ping" target="_blank">http://phonehome.hazelcast.com/ping</a>.

