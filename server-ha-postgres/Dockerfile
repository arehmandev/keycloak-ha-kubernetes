FROM jboss/keycloak-postgres:latest

CMD ["-b", "0.0.0.0", "--server-config", "standalone-ha.xml"]

RUN rm keycloak/standalone/configuration/standalone.xml

COPY logging.properties keycloak/standalone/configuration/
COPY standalone-ha.xml keycloak/standalone/configuration/
