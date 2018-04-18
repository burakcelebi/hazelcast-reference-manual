
A failure detector is responsible to determine if a member in the cluster is unreachable or crashed. The most important problem in failure detection is to distinguish whether a member is still alive but slow or has crashed. But according to the famous [FLP result](http://dl.acm.org/citation.cfm?doid=3149.214121), it is impossible to distinguish a crashed member from a slow one in an asynchronous system. A workaround to this limitation is to use unreliable failure detectors. An unreliable failure detector allows a member to suspect that others have failed, usually based on liveness criteria but it can make mistakes to a certain degree.

Hazelcast has two built-in failure detectors; _Deadline Failure Detector_ and _Phi (ϕ) Accrual Failure Detector_.

### Deadline Failure Detector

_Deadline Failure Detector_ uses an absolute timeout for missing/lost heartbeats. After timeout, a member is considered as crashed/unavailable and marked as suspected.

_Deadline Failure Detector_ has two configuration properties:

- `hazelcast.heartbeat.interval.seconds`: This is the interval at which member heartbeat messages are sent to each other.
- `hazelcast.max.no.heartbeat.seconds`: This is the timeout which defines when a cluster member is suspected because it has not sent any heartbeats.

To use _Deadline Failure Detector_ configuration property `hazelcast.heartbeat.failuredetector.type` should be set to `"deadline"`. 

Please see the following declarative and programmatic configuration examples:

```xml
<hazelcast>
    [...]
	<properties>
        <property name="hazelcast.heartbeat.failuredetector.type">deadline</property>
        <property name="hazelcast.heartbeat.interval.seconds">5</property>
        <property name="hazelcast.max.no.heartbeat.seconds">120</property>
        [...]
    </properties>
    [...]
</hazelcast>
```

```java
Config config = ...;
config.setProperty("hazelcast.heartbeat.failuredetector.type", "deadline");
config.setProperty("hazelcast.heartbeat.interval.seconds", "5");
config.setProperty("hazelcast.max.no.heartbeat.seconds", "120");
[...]
```

![image](../images/NoteSmall.jpg) ***NOTE:*** _Deadline Failure Detector_ is the default failure detector in Hazelcast.

### ϕ (Phi) Accrual Failure Detector

This is the failure detector based on [The ϕ Accrual Failure Detector' by Hayashibara et al.](https://www.computer.org/csdl/proceedings/srds/2004/2239/00/22390066-abs.html)

_ϕ (Phi) Accrual Failure Detector_ keeps track of the intervals between heartbeats in a sliding window of time and measures the mean and variance of these samples and calculates a value of suspicion level (ϕ). The value of ϕ (phi) will increase when the period since the last heartbeat gets longer. If the network becomes slow or unreliable, the resulting mean and variance will increase, there will need to be a longer period for which no heartbeat is received before the member is suspected. 

`hazelcast.heartbeat.interval.seconds` and `hazelcast.max.no.heartbeat.seconds` properties will still be used as period of heartbeat messages and deadline of heartbeat messages. Since _ϕ (Phi) Accrual Failure Detector_ is adaptive to network conditions, a much lower `hazelcast.max.no.heartbeat.seconds` can be defined than _Deadline Failure Detector_'s timeout.

Additional to above two properties, _ϕ (Phi) Accrual Failure Detector_ has three more configuration properties:

- `hazelcast.heartbeat.phiaccrual.failuredetector.threshold`: This is the ϕ threshold for suspicion. After calculated ϕ (phi) exceeds this threshold, a member is considered as unreachable and marked as suspected. A low threshold allows to detect member crashes/failures faster but can generate more mistakes and cause wrong member suspicions. A high threshold generates fewer mistakes but is slower to detect actual crashes/failures.

 `ϕ = 1` means likeliness that we will make a mistake is about `10%`. The likeliness is about `1%` with `ϕ = 2`, `0.1%` with `ϕ = 3`, and so on. Default ϕ (phi) threshold is 10.

- `hazelcast.heartbeat.phiaccrual.failuredetector.sample.size`: Number of samples to keep for history. Default value is 200.

- `hazelcast.heartbeat.phiaccrual.failuredetector.min.std.dev.millis`: Minimum standard deviation to use for the normal distribution used when calculating ϕ (phi). Too low standard deviation might result in too much sensitivity.

To use _ϕ (Phi) Accrual Failure Detector_ configuration property `hazelcast.heartbeat.failuredetector.type` should be set to `"phi-accrual"`.

Please see the following declarative and programmatic configuration examples:

```xml
<hazelcast>
    [...]
	<properties>
        <property name="hazelcast.heartbeat.failuredetector.type">phi-accrual</property>
        <property name="hazelcast.heartbeat.interval.seconds">1</property>
        <property name="hazelcast.max.no.heartbeat.seconds">60</property>
        <property name="hazelcast.heartbeat.phiaccrual.failuredetector.threshold">10</property>
        <property name="hazelcast.heartbeat.phiaccrual.failuredetector.sample.size">200</property>
        <property name="hazelcast.heartbeat.phiaccrual.failuredetector.min.std.dev.millis">100</property>
        [...]
    </properties>
    [...]
</hazelcast>
```

```java
Config config = ...;
config.setProperty("hazelcast.heartbeat.failuredetector.type", "phi-accrual");
config.setProperty("hazelcast.heartbeat.interval.seconds", "1");
config.setProperty("hazelcast.max.no.heartbeat.seconds", "60");
config.setProperty("hazelcast.heartbeat.phiaccrual.failuredetector.threshold", "10");
config.setProperty("hazelcast.heartbeat.phiaccrual.failuredetector.sample.size", "200");
config.setProperty("hazelcast.heartbeat.phiaccrual.failuredetector.min.std.dev.millis", "100");
[...]
```


### Ping Failure Detector

The Ping Failure Detector may be configured in addition to one of Deadline and Phi Accual Failure Detectors. It operates at Layer 3 of the OSI protocol, and provides much quicker and more deterministic detection of hardware and other lower level events. This detector may be configured to perform an extra check after a member is suspected by one of the other detectors, or it can work in parallel, which is the default. This way hardware and network level issues will be detected more quickly.  

This failure detector is based on `InetAddress.isReachable()`.
When the JVM process has enough permissions to create RAW sockets, the implementation will choose to rely on ICMP Echo requests. This is preferred.

If there are not enough permissions, it can be configured to fallback on attempting a TCP Echo on port 7. In the latter case, both a successful connection or an explicit rejection will be treated as "Host is Reachable". Or, it can be forced to use only RAW sockets. This is not preferred as each call creates a heavy weight socket and moreover the Echo service is typically disabled. 

For the Ping Failure Detector to rely **only** on ICMP Echo requests, there are some criteria that need to be met.

#### Requirements and Linux/Unix Configuration

- **Supported OS: as of Java 1.8 only Linux/Unix environments are supported**. This detector relies on ICMP, i.e., the protocol behind the `ping` command. It tries to issue the ping attempts periodically, and their responses are used to determine the reachability of the remote member. However, you cannot simply create an ICMP Echo Request because these type of packets do not rely on any of the preexisting transport protocols such as TCP. In order to create such a request, you must have the privileges to create RAW sockets (please see [https://linux.die.net/man/7/raw](https://linux.die.net/man/7/raw)). Most operating systems allow this to the root users, however Unix based ones are more flexible and allow the use of custom privileges per process
instead of requiring root access. Therefore, this detector is supported only on Linux.

- **The Java executable must have the `cap_net_raw` capability.** As described in the above requirement, on Linux, you have the ability to define extra capabilities to a single process, which would allow the process to interact with the RAW sockets. This interaction is achieved via the capability `cap_net_raw` (please see [https://linux.die.net/man/7/capabilities](https://linux.die.net/man/7/capabilities)). To enable this capability run the following command:

    - `sudo setcap cap_net_raw=+ep <JDK_HOME>/jre/bin/java`

- **When running with custom capabilities, the dynamic linker on Linux will reject loading libs from untrusted paths.** Since you have now `cap_net_raw` as a custom capability for a process, it becomes suspicious to the dynamic linker and it will throw an error:
 `java: error while loading shared libraries: libjli.so: cannot open shared object file: No such file or directory`
    - To overcome this rejection, the `<JDK_HOME>/jre/lib/amd64/jli/` path needs to be added in the `ld.conf`. Run the following command to do this: `echo "<JDK_HOME>/jre/lib/amd64/jli/" >> /etc/ld.so.conf.d/java.conf && sudo ldconfig`

- **ICMP Echo Requests must not be blocked by the receiving hosts.** `/proc/sys/net/ipv4/icmp_echo_ignore_all` set to `0`. Run the following command:
    - `echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all`


If any of the above criteria isn't met, then the `isReachable` will always fallback on TCP Echo attempts on port 7.

To be able to use the Ping Failure Detector, please add the following properties in your Hazelcast declarative configuration file:

```xml
<hazelcast>
    [...]
	<properties>
        <property name="hazelcast.icmp.enabled">true</property>
        <property name="hazelcast.icmp.parallel.mode">true</property>
        <property name="hazelcast.icmp.timeout">1000</property>
        <property name="hazelcast.icmp.max.attempts">3</property>
        <property name="hazelcast.icmp.interval">1000</property>
        <property name="hazelcast.icmp.ttl">0</property>
        [...]
    </properties>
    [...]
</hazelcast>
```

- `hazelcast.icmp.enabled` (default false) - Enables legacy ICMP detection mode, works cooperatively with the existing failure detector, and only kicks-in after a pre-defined period has passed with no heartbeats from a member.
- `hazelcast.icmp.parallel.mode` (default true) - Enabling the parallel ping detector, works separately from the other detectors.
- `hazelcast.icmp.timeout` (default 1000) - Number of milliseconds until a ping attempt is considered failed if there was no reply.
- `hazelcast.icmp.max.attempts` (default 3) - The maximum number of ping attempts before the member/node gets suspected by the detector.
- `hazelcast.icmp.interval` (default 1000) - The interval, in milliseconds, between each ping attempt. 1000ms (1 sec) is also the minimum interval allowed.
- `hazelcast.icmp.ttl` (default 0) - The maximum number of hops the packets should go through or 0 for the default.

In the above configuration, the Ping detector will attempt 3 pings, one every second and will wait up-to 1 second for each to complete. If after 3 seconds, there was no successful ping, the member will get suspected.

To enforce the requirements stated above, the property `hazelcast.icmp.echo.fail.fast.on.startup` can also be set to `true`, in which case, if any of the requirements
isn't met, Hazelcast will fail to start.

Below is a summary table of all possible configuration combinations of the ping failure detector.

| ICMP  	| Parallel 	| Fail-Fast 	| Description                                                                                                                                                                                                                                                                                                                                   | Linux                                                                                                  	| Windows                       	| macOS                                                                	|
|-------	|----------	|-----------	|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |--------------------------------------------------------------------------------------------------------	|-------------------------------	|----------------------------------------------------------------------	|
| false 	| false    	| false     	| Completely disabled                                                                                                                                                                                                         	                                                                                                                | N/A                                                                                                    	| N/A                           	| N/A                                                                  	|
| true  	| false    	| false     	| Legacy ping mode. This works hand-to-hand with the OSI Layer 7 failure detector (see. phi or deadline in sections above). Ping in this mode will only kick in after a period when there are no hearbeats received, in which case the remote Hazelcast member will be pinged up-to 5 times. If all 5 attempts fail, the member gets suspected. | Supported  ICMP Echo if available - Falls back on TCP Echo on port 7                                   	| Supported  TCP Echo on port 7 	|  Supported ICMP Echo if available - Falls back on TCP Echo on port 7 	|
| true  	| true     	| false     	| Parallel ping detector, works in parallel with the configured failure detector. Checks periodically if members are live (OSI Layer 3), and suspects them immediately, regardless of the other detectors.                      	                                                                                                            | Supported  ICMP Echo if available - Falls back on TCP Echo on port 7                                   	| Supported  TCP Echo on port 7 	| Supported  ICMP Echo if available - Falls back on TCP Echo on port 7 	|
| true  	| true     	| true      	| Parallel ping detector, works in parallel with the configured failure detector. Checks periodically if members are live (OSI Layer 3), and suspects them immediately, regardless of the other detectors.                      	                                                                                                            | Supported - Requires OS Configuration  Enforcing ICMP Echo if available - No start up if not available 	| Not Supported                 	| Not Supported - Requires root privileges                            	|

