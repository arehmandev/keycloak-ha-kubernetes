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
