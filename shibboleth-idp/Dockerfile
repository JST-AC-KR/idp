# FROM alpine:3.11.6 as temp
FROM amazoncorretto:11 as temp

ENV jetty_version=9.4.34.v20201102 \
    jetty_hash=10a7d765abd2709ce544f09100650a48 \
    #idp_version=3.4.8 \
    #idp_hash=ad0fcd834d0c6571363d47ad6dde08fbb75cce3202c41f8c64a5b42614f95a27 \
    idp_version=4.2.1 \
    idp_hash=fa5e46d160f6b1bc50326c1a31627a05b5d0847b8f620d7f4c0251999b806474 \
    idp_oidcext_version=2.0.0 \
    idp_oidcext_hash=304eb4e58eadc3377fae02209f8eef6549fd17ac5fd9356ad1216869b75bb23a \
    slf4j_version=1.7.29 \
    slf4j_hash=47b624903c712f9118330ad2fb91d0780f7f666c3f22919d0fc14522c5cad9ea \
    logback_version=1.2.3 \
    logback_classic_hash=fb53f8539e7fcb8f093a56e138112056ec1dc809ebb020b59d8a36a5ebac37e0 \
    logback_core_hash=5946d837fe6f960c02a53eda7a6926ecc3c758bbdd69aa453ee429f858217f22 \
    logback_access_hash=0a4fc8753abe266ea7245e6d9653d6275dc1137cad6ecd1b2612204033d89687 \
    mariadb_version=2.5.4 \
    mariadb_hash=5fafee1aad82be39143b4bfb8915d6c2d73d860938e667db8371183ff3c8500a

ARG IDP_SCOPE 
ARG IDP_SCOPE_DOMAIN 
ARG IDP_HOST_NAME 
ARG IDP_ENTITYID 
ARG IDP_KEYSTORE_PASSWORD 
ARG IDP_SEALER_PASSWORD 

ENV JETTY_HOME=/opt/jetty-home \
    JETTY_BASE=/opt/shib-jetty-base \
    IDP_HOME=/opt/shibboleth-idp \
    IDP_SRC=/opt/shibboleth-identity-provider-$idp_version \
    PATH=$PATH:$JAVA_HOME/bin

RUN echo ${IDP_HOST_NAME}
LABEL maintainer="chrisryu"\
      idp.java.version="Alpine - openjdk11-jre-headless" \
      idp.jetty.version=$jetty_version \
      idp.version=$idp_version

RUN yum install -y wget tar bash gawk gzip shadow-utils

# JETTY - Download, verify and install with base
RUN wget -q https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-distribution/$jetty_version/jetty-distribution-$jetty_version.tar.gz \
    && echo "$jetty_hash  jetty-distribution-$jetty_version.tar.gz" | md5sum -c - \
    && tar -zxvf jetty-distribution-$jetty_version.tar.gz -C /opt \
    && ln -s /opt/jetty-distribution-$jetty_version/ /opt/jetty-home \
    && rm jetty-distribution-$jetty_version.tar.gz

# JETTY Configure
RUN mkdir -p $JETTY_BASE/modules $JETTY_BASE/lib/ext $JETTY_BASE/lib/logging $JETTY_BASE/resources
    # && cd $JETTY_BASE \
    # && touch start.ini \
    # && $JAVA_HOME/bin/java -jar $JETTY_HOME/start.jar --create-startd --add-to-start=idp,https,http2,http2c,deploy,ext,annotations,jstl,rewrite,setuid

# Shibboleth IdP - Download, verify hash and install
RUN wget -q https://shibboleth.net/downloads/identity-provider/archive/$idp_version/shibboleth-identity-provider-$idp_version.tar.gz \
    && echo "$idp_hash  shibboleth-identity-provider-$idp_version.tar.gz" | sha256sum -c - \
    && tar -zxvf  shibboleth-identity-provider-$idp_version.tar.gz -C /opt \
    && S=$IDP_SRC/idp.merge.properties \
    && echo idp.scope=${IDP_SCOPE}>>$S \
    && echo idp.scope_domain=${IDP_SCOPE_DOMAIN}>>$S \
    && echo idp.entityID=${IDP_ENTITYID}>>$S \
    && echo idp.status.accessPolicy=status.AccessByIPAddress>>$S \
    && $IDP_SRC/bin/install.sh \
    -Didp.scope=${IDP_SCOPE} \
    -Didp.scope_domain=${IDP_SCOPE_DOMAIN} \
    -Didp.target.dir=$IDP_HOME \
    -Didp.src.dir=$IDP_SRC \
    -Didp.host.name=${IDP_HOST_NAME} \
    -Didp.noprompt=true \
    -Didp.sealer.password=${IDP_SEALER_PASSWORD} \
    -Didp.keystore.password=${IDP_KEYSTORE_PASSWORD} \
    -Didp.entityID=${IDP_ENTITYID} \
    -Didp.merge.properties=idp.merge.properties \
    -Didp.initial.modules=+idp.intercept.Consent \
    && rm shibboleth-identity-provider-$idp_version.tar.gz \
    && rm -rf shibboleth-identity-provider-$idp_version


COPY opt/shibboleth-idp/ $IDP_HOME/

RUN sed -i 's/__IDP_SCOPE__/'${IDP_SCOPE}'/' $IDP_HOME/messages/messages_ko.properties
RUN sed -i 's/__IDP_SCOPE_DOMAIN__/'${IDP_SCOPE_DOMAIN}'/' $IDP_HOME/messages/messages_ko.properties
RUN sed -i 's/__IDP_HOST_NAME__/'${IDP_HOST_NAME}'/' $IDP_HOME/messages/messages_ko.properties
RUN sed -i 's/__IDP_ORG_HOMEPAGE__ /'${IDP_ORG_HOMEPAGE}'/' $IDP_HOME/messages/messages_ko.properties
RUN sed -i 's/__IDP_ORG_DISPLAYNAME__ /'${IDP_ORG_DISPLAYNAME}'/' $IDP_HOME/messages/messages_ko.properties
RUN sed -i 's/__IDP_FORGOT_PASSWORD_URL__ /'${IDP_FORGOT_PASSWORD_URL}'/' $IDP_HOME/messages/messages_ko.properties
RUN sed -i 's/__IDP_SUPPORT_URL__ /'${IDP_SUPPORT_URL}'/' $IDP_HOME/messages/messages_ko.properties


COPY $JETTY_BASE $JETTY_BASE
RUN sed -i 's/changeme/'${IDP_KEYSTORE_PASSWORD}'/' $JETTY_BASE/start.d/idp.ini
# Create new user to run jetty with
# RUN addgroup -g 1000 -S jetty && \
# RUN adduser -u 1000 -S jetty -G root -s /sbin/nologin
# RUN useradd jetty

RUN useradd --system --no-create-home --user-group jetty

# Set ownerships
RUN mkdir -p $JETTY_BASE/logs \
    && chown -R jetty:jetty $IDP_HOME \
    && chown -R jetty:jetty $JETTY_BASE \
    && chmod -R 550 $IDP_HOME \
    && chmod -R 775 $IDP_HOME/metadata

FROM amazoncorretto:11

RUN yum install -y bash curl shadow-utils which tar gzip


LABEL maintainer="chrisryu"\
    idp.java.version="Alpine - openjdk11-jre-headless" \
    idp.jetty.version=$jetty_version \
    idp.version=$idp_version

COPY bin/ /usr/local/bin/

#RUN addgroup -g 1000 -S jetty \
#    && adduser -u 1000 -S jetty -G jetty -s /bin/false \
#    && chmod 750 /usr/local/bin/run-jetty.sh /usr/local/bin/init-idp.sh

# RUN adduser -u 1000 -S jetty -G root -s /sbin/nologin \
RUN useradd jetty \
    && chmod 750 /usr/local/bin/init-idp.sh

COPY --from=temp /opt/ /opt/

RUN /opt/shibboleth-idp/bin/build.sh -Didp.target.dir=/opt/shibboleth-idp
RUN chmod +x /opt/jetty-home/bin/jetty.sh

# Opening 8080
EXPOSE 443 80
ENV JETTY_HOME=/opt/jetty-home \
    JETTY_BASE=/opt/shib-jetty-base \
    PATH=$PATH:$JAVA_HOME/bin
    # JAVA_HOME=/usr/lib/jvm/default-jvm \

#establish a healthcheck command so that docker might know the container's true state
HEALTHCHECK --interval=1m --timeout=30s \
    CMD curl -k -f http://localhost/idp/status || exit 1


CMD /opt/jetty-home/bin/jetty.sh run
#RUN /opt/shibboleth-idp/bin/plugin.sh --noPrompt -I net.shibboleth.idp.plugin.metadatagen
