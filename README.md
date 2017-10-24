ssls


``` bash
git submodule add github.com:oondeo/docker-scripts-root.git root
mkdir -p s2i/bin
```

``` Dockerfile

#At the init of file
ENV SUMMARY="PHP 5 image with standar modules"	\
    DESCRIPTION="The image use scripts and configurations compatible \
        with redhat openshift."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="php 5" \
      io.openshift.s2i.scripts-url=image:///usr/libexec/s2i \
      io.s2i.scripts-url=image:///usr/libexec/s2i \
      com.redhat.component="php" \
      name="oondeo/php" \
      version="5" \
      release="6" \
      maintainer="OONDEO <info@oondeo.es>"

ENV \
    # DEPRECATED: Use above LABEL instead, because this will be removed in future versions.
    STI_SCRIPTS_URL=image:///usr/libexec/s2i \
    # Path to be used in other layers to place s2i scripts into
    STI_SCRIPTS_PATH=/usr/libexec/s2i \
    # The $HOME is not set by default, but some applications needs this variable
    HOME=/opt/app-root/src \
    PATH=/opt/app-root/src/bin:/opt/app-root/bin:$PATH


# When bash is started non-interactively, to run a shell script, for example it
# looks for this variable and source the content of this file. This will enable
# the SCL for all scripts without need to do 'scl enable'.
ENV BASH_ENV=/opt/app-root/etc/scl_enable \
    ENV=/opt/app-root/etc/scl_enable \
    XDG_DATA_HOME=$HOME/.local/share \
    DEBIAN_FRONTEND=noninteractive \
    LANG=es_ES.UTF-8 LANGUAGE=es_ES.UTF-8 LC_ALL=es_ES.UTF-8 \
    PROMPT_COMMAND=". /opt/app-root/etc/scl_enable"

# Copy extra files to the image.
COPY ./root/ /
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

#Install tiny on debian 
ENV TINI_VERSION=v0.13.0
RUN cp /bin/sh /sh && apt-get update && apt-get install -y --no-install-recommends curl dash \
    && curl -o /sbin/tini -SLk https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini \
    && chmod +x /sbin/tini && rm -rf /var/lib/apt/* /var/tmp/* /tmp/* /var/log/* 

# Directory with the sources is set as the working directory so all STI scripts
# can execute relative to this path.
WORKDIR ${HOME}

ENTRYPOINT ["/sbin/tini", "-g" ,"--", "container-entrypoint"]

# Last line in dockerfile
# Reset permissions of modified directories and add default user
RUN useradd -u 1001 -r -g 0 -d ${HOME} -s /sbin/nologin \
      -c "Default Application User" default \
    && chown -R 1001:0 /opt/app-root \
    && mkdir -p ${HOME}/.pki/nssdb \
    && chown -R 1001:0 ${HOME}/.pki     
USER 1001

```