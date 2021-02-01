FROM debian:buster-slim
ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /setup-files

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    wget \
    ca-certificates \
    gnupg && \
    rm -rf /var/lib/apt/lists/*

RUN wget -q -O - https://apt.mopidy.com/mopidy.gpg | apt-key add - && \
    wget -q -O /etc/apt/sources.list.d/mopidy.list https://apt.mopidy.com/buster.list

RUN wget -q -O /usr/share/keyrings/lesbonscomptes.gpg https://www.lesbonscomptes.com/pages/lesbonscomptes.gpg  && \
    wget -q -O /etc/apt/sources.list.d/upmpdcli-buster.list https://www.lesbonscomptes.com/upmpdcli/pages/upmpdcli-buster.list


RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    tzdata \
    sudo \
    build-essential \
    python3-dev \
    python3-pip \
    python3-gst-1.0 \
    python3-wheel \
    gir1.2-gstreamer-1.0 \
    gir1.2-gst-plugins-base-1.0 \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-libav \
    gstreamer1.0-tools \
    libspotify12 \
    libspotify-dev \
    libxml2-dev \
    libxslt1-dev \
    libffi-dev \
    libz-dev \
    libdbus-glib-1-dev \
    upmpdcli \
    dleyna-server \
    python3-setuptools \
    python3-spotify && \
    rm -rf /var/lib/apt/lists/*

RUN set -ex \
 && apt-get update && apt-get install -y --no-install-recommends avahi-daemon libnss-mdns supervisor \
 # allow hostnames with more labels to be resolved. so that we can
 # resolve node1.mycluster.local.
 # (https://github.com/lathiat/nss-mdns#etcmdnsallow)
 && echo '*' > /etc/mdns.allow \
 # Configure NSSwitch to use the mdns4 plugin so mdns.allow is respected
 && sed -i "s/hosts:.*/hosts:          files mdns4 dns/g" /etc/nsswitch.conf \
 && printf "[server]\nenable-dbus=no\n" >> /etc/avahi/avahi-daemon.conf \
 && chmod 777 /etc/avahi/avahi-daemon.conf \
 && mkdir -p /var/run/avahi-daemon \
 && chown avahi:avahi /var/run/avahi-daemon \
 && chmod 777 /var/run/avahi-daemon


RUN pip3 install \
    Mopidy \
    Mopidy-MPD \
    Mopidy-Local \
    Mopidy-TuneIn \
    Mopidy-dLeyna \
    Mopidy-Spotify \
    Mopidy-YouTube \
    dbus-python \
    Mopidy-Iris

RUN mkdir -p /mopidy
RUN touch /IS_CONTAINER
RUN useradd -ms /bin/bash mopidy
RUN chown mopidy:mopidy /mopidy
RUN sh -c 'echo "mopidy ALL=NOPASSWD: /usr/local/lib/python3.7/dist-packages/mopidy_iris/system.sh, /usr/bin/apt*" >> /etc/sudoers'
RUN sed -i 's+--config .*mopidy.conf+--config /mopidy/mopidy.conf+g' /usr/local/lib/python3.7/dist-packages/mopidy_iris/system.sh

ADD setup-files/*.conf /setup-files/ 
ADD setup-files/*.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/*.sh 
#COPY supervisord.conf /supervisord.conf
#COPY mopidy.conf /mopidy_default.conf
#COPY mopidy.sh /usr/local/bin/mopidy.sh
#COPY upmpdcli.sh /usr/local/bin/upmpdcli.sh

EXPOSE 6600 6680 1900/udp 49152
USER mopidy

#ENTRYPOINT ["/usr/local/bin/mopidy.sh"]
CMD ["/usr/bin/supervisord", "-c", "/setup-files/supervisord.conf"]
#CMD ["/bin/bash"]
