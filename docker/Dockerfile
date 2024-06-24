FROM debian:bookworm-slim

# Install host toolchain
ENV DEBIAN_FRONTEND noninteractive
RUN set -e; \
	apt-get update; \
	apt-get upgrade -y; \
	apt-get install -y build-essential ca-certificates git locales wget; \
	locale-gen en_GB.UTF-8; \
	update-ca-certificates; \
	:

# set locales
ENV \
	LANG=en_GB.UTF-8 \
	LANGUAGE=en_GB:en \
	LC_ALL=en_GB.UTF-8

# install cross toolchain
RUN set -e; \
	wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu/13.2.rel1/binrel/arm-gnu-toolchain-13.2.rel1-x86_64-arm-none-eabi.tar.xz; \
	tar -C /opt -xf arm-gnu-toolchain-13.2.rel1-x86_64-arm-none-eabi.tar.xz; \
	rm -f arm-gnu-toolchain-13.2.rel1-x86_64-arm-none-eabi.tar.xz; \
	:
ENV PATH="/opt/arm-gnu-toolchain-13.2.Rel1-x86_64-arm-none-eabi/bin:$PATH"

# install "newt" tool
RUN set -e; \
	wget https://archive.apache.org/dist/mynewt/apache-mynewt-1.11.0/apache-mynewt-newt-bin-linux-1.11.0.tgz; \
	tar --strip-components=1 -xvf apache-mynewt-newt-bin-linux-1.11.0.tgz apache-mynewt-newt-bin-linux-1.11.0/newt; \
	install -v -m755 -o root -g root newt /usr/local/bin/; \
	rm newt; \
	:

# Arguments to pass host user's UID and GID
ARG USER_ID=1000
ARG GROUP_ID=1000

# Create a user 'developer' with the same UID/GID as the host user
RUN groupadd -g ${GROUP_ID} developer
RUN useradd -m -u ${USER_ID} -g developer -d /home/developer developer
RUN mkdir -p /workspace
RUN chown -R developer:developer /workspace
RUN mkdir -p /home/developer/.ssh
RUN touch /home/developer/.ssh/known_hosts
RUN chmod 700 /home/developer/.ssh
RUN chmod 600 /home/developer/.ssh/known_hosts
RUN chown -R developer:developer /home/developer/.ssh

RUN git config --add --system user.email "developer@example.com"
RUN git config --add --system user.name "Build Container"
RUN git config --add --system http.version HTTP/1.1

# The entry point is set to bash. This means that when the container starts, it will drop the user into bash
ENTRYPOINT ["/bin/bash"]
