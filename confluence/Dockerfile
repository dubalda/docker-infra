FROM nearpod/alpine-jvm:3.7-oracle-jdk8

ARG CONF_HOME=/var/atlassian/confluence
ARG CONF_INSTALL=/opt/atlassian/confluence
ARG CONF_VERSION=6.9.1
ARG JDBC_MYSQL_VERSION=5.1.44
ARG JDBC_POSTGRES_VERSION=42.1.4

ENV CONF_VERSION=$CONF_VERSION \
    CONF_HOME=$CONF_HOME \
    CONF_INSTALL=$CONF_INSTALL \
    JAVA_CACERTS=$JAVA_HOME/jre/lib/security/cacerts

RUN set -ex; \
  apk --no-cache add xmlstarlet apr openssl; \
  apk --no-cache fetch tomcat-native; \
  apk add -f ./tomcat-native-*.apk; \
  rm -f ./tomcat-native-*.apk; \
  mkdir -p "${CONF_HOME}"; \
  mkdir -p "${CONF_INSTALL}/conf/Catalina"; \
  curl -q -sSL "https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-${CONF_VERSION}.tar.gz" | tar -xz --directory "${CONF_INSTALL}" --strip-components=1 --no-same-owner; \
  curl -q -sSLO "http://central.maven.org/maven2/mysql/mysql-connector-java/$JDBC_MYSQL_VERSION/mysql-connector-java-$JDBC_MYSQL_VERSION.jar"; \
  curl -q -sSLO "http://central.maven.org/maven2/mysql/mysql-connector-java/$JDBC_MYSQL_VERSION/mysql-connector-java-$JDBC_MYSQL_VERSION.jar.md5"; \
  echo "$(cat mysql-connector-java-$JDBC_MYSQL_VERSION.jar.md5)  mysql-connector-java-$JDBC_MYSQL_VERSION.jar" | md5sum -c; \
  mkdir -p "$KEYCLOAK_HOME/modules/system/layers/base/com/mysql/jdbc/main/"; \
  mv "mysql-connector-java-$JDBC_MYSQL_VERSION.jar" "${CONF_INSTALL}/lib/"; \
  curl -q -sSLO "http://central.maven.org/maven2/org/postgresql/postgresql/$JDBC_POSTGRES_VERSION/postgresql-$JDBC_POSTGRES_VERSION.jar"; \
  curl -q -sSLO "http://central.maven.org/maven2/org/postgresql/postgresql/$JDBC_POSTGRES_VERSION/postgresql-$JDBC_POSTGRES_VERSION.jar.md5"; \
  echo "$(cat postgresql-$JDBC_POSTGRES_VERSION.jar.md5)  postgresql-$JDBC_POSTGRES_VERSION.jar" | md5sum -c; \
  rm -f "${CONF_INSTALL}/lib/postgresql-9.1-903.jdbc4-atlassian-hosted.jar"; \
  mv "postgresql-$JDBC_POSTGRES_VERSION.jar" "${CONF_INSTALL}/lib/postgresql-${POSTGRES_JDBC_VERSION}.jar"; \
  mkdir -p "${CONF_HOME}"; \
  rm -f "mysql-connector-java-$JDBC_MYSQL_VERSION.jar.md5" "postgresql-$JDBC_POSTGRES_VERSION.jar.md5"; \
  chmod -R 700 "${CONF_INSTALL}/" "${CONF_HOME}/"; \
  chown -R daemon:daemon  "${CONF_INSTALL}/" "${CONF_HOME}"; \
  echo -e "\nconfluence.home=$CONF_HOME" >> "${CONF_INSTALL}/confluence/WEB-INF/classes/confluence-init.properties"; \
  xmlstarlet              ed --inplace \
    --delete               "Server/@debug" \
    --delete               "Server/Service/Connector/@debug" \
    --delete               "Server/Service/Connector/@useURIValidationHack" \
    --delete               "Server/Service/Connector/@minProcessors" \
    --delete               "Server/Service/Connector/@maxProcessors" \
    --delete               "Server/Service/Engine/@debug" \
    --delete               "Server/Service/Engine/Host/@debug" \
    --delete               "Server/Service/Engine/Host/Context/@debug" \
                           "${CONF_INSTALL}/conf/server.xml" ; \
  touch -d "@0" "${CONF_INSTALL}/conf/server.xml" ; \
  chown daemon:daemon     "${JAVA_CACERTS}"

USER daemon:daemon

# Expose default HTTP connector port.
EXPOSE 8090 8091

# Set volume mount points for installation and home directory. Changes to the
# home directory needs to be persisted as well as parts of the installation
# directory due to eg. logs.
VOLUME ["/var/atlassian/confluence", "/opt/atlassian/confluence/logs"]

# Set the default working directory as the installation directory.
WORKDIR /var/atlassian/confluence

COPY "docker-entrypoint.sh" "/"
ENTRYPOINT ["/docker-entrypoint.sh"]

# Run Atlassian CONFLUENCE as a foreground process by default.
CMD ["/opt/atlassian/confluence/bin/start-confluence.sh", "-fg"]