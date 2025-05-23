FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    sudo \
    git \
    tig \
    bash-completion \
    python3 \
    python3-pip \
    vim \
    curl \
    locales \
    bc \
    iputils-ping \
    wget \
    nmap \
    && rm -rf /var/lib/apt/lists/*

RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen \
    && locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN python3 -m pip install pymodbus flask asyncua nicegui pytest pytest-asyncio

RUN install -d -m 0755 /etc/apt/keyrings && \
    wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- > /etc/apt/keyrings/packages.mozilla.org.asc && \
    echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" > /etc/apt/sources.list.d/mozilla.list && \
    echo 'Package: *' > /etc/apt/preferences.d/mozilla && \
    echo 'Pin: origin packages.mozilla.org' >> /etc/apt/preferences.d/mozilla && \
    echo 'Pin-Priority: 1000' >> /etc/apt/preferences.d/mozilla && \
    apt-get update && apt-get install -y firefox

RUN echo "ALL ALL=(ALL) NOPASSWD: ALL" >/etc/sudoers.d/nopasswd

RUN addgroup --gid 1000 docker \
    && adduser --uid 1000 --ingroup docker --home /home/docker --disabled-password --gecos "" docker \
    && echo 'docker:docker' | chpasswd \
    && usermod -a -G sudo docker

RUN USER=docker \
    && GROUP=docker \
    && curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.6.0/fixuid-0.6.0-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - \
    && chown root:root /usr/local/bin/fixuid \
    && chmod 4755 /usr/local/bin/fixuid \
    && mkdir -p /etc/fixuid \
    && printf "user: $USER\ngroup: $GROUP\n" >/etc/fixuid/config.yml

# Create docker-init.sh
RUN echo "#!/bin/sh\n\
    fixuid \n\
    \"\$@\"" >>/usr/local/share/docker-init.sh \
    && chmod +x /usr/local/share/docker-init.sh

ENV HOME /home/docker
USER docker:docker
# VS Code overrides ENTRYPOINT and CMD when executing `docker run` by default.
# Setting the ENTRYPOINT to docker-init.sh will configure non-root access to
# the Docker socket if "overrideCommand": false is set in devcontainer.json.
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]
