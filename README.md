# Keycloak Docker image

This is a set of Docker images related to Keycloak. 

- [keycloak](https://registry.hub.docker.com/u/jboss/keycloak/) example Keycloak server
- [keycloak-mysql](https://registry.hub.docker.com/u/jboss/keycloak-mysql/) example connecting Keycloak to MySQL
- [keycloak-postgres](https://registry.hub.docker.com/u/jboss/keycloak-postgres/) example connecting Keycloak to PostgreSQL
- [keycloak-adapter-wildfly](https://registry.hub.docker.com/u/jboss/keycloak-adapter-wildfly/) builds on top of the [jboss/wildfly](https://registry.hub.docker.com/u/jboss/wildfly/) image, adding the Keycloak adapter for Wildfly to it, as well as the required changes to the standalone.xml
- [keycloak-examples](https://registry.hub.docker.com/u/jboss/keycloak-examples/) builds on top of [keycloak](https://registry.hub.docker.com/u/jboss/keycloak/), adding the examples.

## Keycloak HA on Kubernetes

Keycloak 2.3.0 runs [JGroups 4](http://jgroups.org/manual4/index.html) with [Infinispan 8.1](http://infinispan.org/docs/8.1.x/index.html) as subsystems to [Wildfly 10](https://docs.jboss.org/author/display/WFLY10/High+Availability+Guide).

What hostname do your pods think they have? Can be changed using system properties.
```
kubectl logs keycloak-0-[tab] | grep "keycloak-0"
```

What physical address does [Infinispan](http://infinispan.org/) think it has. Should we change it to the service's address?
```
kubectl logs keycloak-0-[tab] | grep "nfinispan" | grep "address"
kubectl exec keycloak-0-[tab] -- env | grep KEYCLOAK_0_SERVICE_HOST
```

Why do you get connection refused on all listed ports except 8080 and 54200? And is it because of UDP that 54200 never responds?
```
kubectl exec testpod -- curl http://keycloak-0:54200/
```

What's the role of the node identifier? "Please make sure it is unique"
```
kubectl logs keycloak-0-[tab] | grep "ode identifier"
```

How can we make keycloak fail after a timeout if cluster size is <N nodes?

Can we simplify discovery if we know the number of nodes and have an env var with the IP of nodes?
https://kb.novaordis.com/index.php/WildFly_Clustering_without_Multicast

Can we bind to all interfaces using `-bprivate=0.0.0.0`? Then set bind_address to the service's address in TCP?
http://jgroups.1086181.n5.nabble.com/JGroups-TCP-over-WAN-td3533.html

Is it possible to remove the validation in JGoups:
```
WARN  [org.jboss.as.clustering.jgroups] (MSC service thread 1-1) WFLYCLJG0006: property bind_addr for protocol org.jgroups.protocols.TCP attempting to override socket binding value 0.0.0.0 : property value 10.0.0.170 will be ignored

ERROR [org.jboss.msc.service.fail] (MSC service thread 1-1) MSC000001: Failed to start service jboss.jgroups.channel.ee: org.jboss.msc.service.StartException in service jboss.jgroups.channel.ee: java.security.PrivilegedActionException: java.net.BindException: [TCP] /0.0.0.0 is not a valid address on any local network interface
	at org.wildfly.clustering.jgroups.spi.service.ChannelBuilder.start(ChannelBuilder.java:80)
Caused by: java.security.PrivilegedActionException: java.net.BindException: [TCP] /0.0.0.0 is not a valid address on any local network interface
	at org.wildfly.security.manager.WildFlySecurityManager.doChecked(WildFlySecurityManager.java:640)
Caused by: java.net.BindException: [TCP] /0.0.0.0 is not a valid address on any local network interface
	at org.jgroups.util.Util.checkIfValidAddress(Util.java:3522)
	at org.jgroups.stack.Configurator.ensureValidBindAddresses(Configurator.java:903)
	at org.jgroups.stack.Configurator.setupProtocolStack(Configurator.java:118)
```

Trying to sort things out through mailing list threads:
http://lists.jboss.org/pipermail/keycloak-user/2016-November/thread.html#8267
https://sourceforge.net/p/javagroups/mailman/javagroups-users/ (first message for Nov 2016)
