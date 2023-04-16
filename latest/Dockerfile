
#
# Setup Stage: install apps
#
# This is a dedicated stage so that donwload archives don't end up on 
# production image and consume unnecessary space.
#

FROM ubuntu:22.04 as setup

ENV IB_GATEWAY_VERSION=10.22.1h
ENV IB_GATEWAY_RELEASE_CHANNEL=latest
ENV IBC_VERSION=3.16.2
ENV DEBIAN_FRONTEND=noninteractive
ENV DEBCONF_NOWARNINGS="yes"

# Prepare system
RUN apt-get update -y
RUN apt-get install --no-install-recommends --yes \
  curl \
  ca-certificates \
  unzip \
  apt-utils
RUN apt-get upgrade -y

WORKDIR /tmp/setup

# Install IB Gateway
# Use this instead of "RUN curl .." to install a local file:
#COPY ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh .
RUN curl -sSL https://github.com/UnusualAlpha/ib-gateway-docker/releases/download/ibgateway-${IB_GATEWAY_RELEASE_CHANNEL}%40${IB_GATEWAY_VERSION}/ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh \
  --output ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh
RUN curl -sSL https://github.com/UnusualAlpha/ib-gateway-docker/releases/download/ibgateway-${IB_GATEWAY_RELEASE_CHANNEL}%40${IB_GATEWAY_VERSION}/ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh.sha256 \
  --output ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh.sha256
RUN sha256sum --check ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh.sha256
RUN chmod a+x ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh
RUN ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh -q -dir /root/Jts/ibgateway/${IB_GATEWAY_VERSION}
ADD ./config/ibgateway/jts.ini /root/Jts/jts.ini

# Install IBC
RUN curl -sSL https://github.com/IbcAlpha/IBC/releases/download/${IBC_VERSION}/IBCLinux-${IBC_VERSION}.zip --output IBCLinux-${IBC_VERSION}.zip
RUN mkdir /root/ibc
RUN unzip ./IBCLinux-${IBC_VERSION}.zip -d /root/ibc
ADD ./config/ibc/config.ini.tmpl /root/ibc/config.ini.tmpl
ADD ./config/ibc/IBC.jar /root/ibc/IBC.jar

# Copy scripts
ADD ./scripts /root/scripts

#
# Build Stage: build production image
#

FROM ubuntu:22.04

ENV IB_GATEWAY_VERSION=10.22.1h

WORKDIR /opt

# Prepare system
RUN apt-get update -y   
RUN apt-get install --no-install-recommends --yes \
  default-jre \
  xvfb

# Copy files
COPY --from=setup /root/ .
COPY --from=setup /usr/local/i4j_jres/ /usr/local/i4j_jres

# Create the user
ARG USERNAME=twsUser
ARG USERID=1000
RUN groupadd --gid $USERID $USERNAME \
    && useradd --uid $USERID --gid $USERID -m $USERNAME 

# Add permissions \
RUN chmod a+x /opt/ibc/scripts/*.sh
RUN chmod u+x /opt/scripts/*.sh
RUN echo "" > /opt/Jts/launcher.log
RUN chmod a+rw /opt/Jts/launcher.log
USER $USERNAME

# IBC env vars
ENV TWS_MAJOR_VRSN=${IB_GATEWAY_VERSION}
ENV TWS_PATH=/opt/Jts
ENV IBC_PATH=/opt/ibc
ENV IBC_INI=/opt/ibc/config.ini
ENV TWS_USERID=
ENV TWS_PASSWORD=
ENV TWOFA_TIMEOUT_ACTION=exit

# Start run script
ENTRYPOINT ["/opt/scripts/run.sh"]