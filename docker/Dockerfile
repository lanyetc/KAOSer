FROM tomcat:8-jre8

LABEL maintainer="Florian JUDITH <florian.judith.b@gmail.com>"

ENV VERSION=7.7.5

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends openjdk-8-jdk ant git patch xmlstarlet certbot


# Download
RUN cd /tmp && \
    git clone https://github.com/XLab-Tongji/KAOSer.git

# Patch EmbedServlet2
ADD assets/embed2js.patch /tmp/KAOSer/war/plugins/

RUN cd /tmp/KAOSer && \
    patch -p1 war/plugins/embed2js.patch && \
    cd /tmp/KAOSer/etc/build && \
    ant war && \
    cd /tmp/KAOSer/build && \
    unzip /tmp/KAOSer/build/draw.war -d $CATALINA_HOME/webapps/draw
    #cp -rp /tmp/KAOSer/build/draw.war $CATALINA_HOME/webapps/

# Update server.xml to set Draw.io webapp to root
RUN cd $CATALINA_HOME && \
    xmlstarlet ed \
    -P -S -L \
    -i '/Server/Service/Engine/Host/Valve' -t 'elem' -n 'Context' \
    -i '/Server/Service/Engine/Host/Context' -t 'attr' -n 'path' -v '/' \
    -i '/Server/Service/Engine/Host/Context[@path="/"]' -t 'attr' -n 'docBase' -v 'draw' \
    -s '/Server/Service/Engine/Host/Context[@path="/"]' -t 'elem' -n 'WatchedResource' -v 'WEB-INF/web.xml' \
    -i '/Server/Service/Engine/Host/Valve' -t 'elem' -n 'Context' \
    -i '/Server/Service/Engine/Host/Context[not(@path="/")]' -t 'attr' -n 'path' -v '/ROOT' \
    -s '/Server/Service/Engine/Host/Context[@path="/ROOT"]' -t 'attr' -n 'docBase' -v 'ROOT' \
    -s '/Server/Service/Engine/Host/Context[@path="/ROOT"]' -t 'elem' -n 'WatchedResource' -v 'WEB-INF/web.xml' \
    conf/server.xml

# Cleanup
RUN rm -r /var/lib/apt/lists/* && \
    rm -rf \
    /tmp/KAOSer

# Copy docker-entrypoint
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

WORKDIR $CATALINA_HOME

EXPOSE 8080 8443

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["catalina.sh", "run"]