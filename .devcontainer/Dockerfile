# Debian 13 (trixie)
FROM debian:13

ARG DEBIAN_FRONTEND=noninteractive
ARG USERNAME=agungdh
ARG USER_UID=1000
ARG USER_GID=1000

# ---------------------------------------------------------------------------
# Base packages
# ---------------------------------------------------------------------------
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        wget \
        git \
        openssh-client \
        gnupg \
        sudo \
        build-essential \
        pkg-config \
        make \
        jq \
        unzip \
        zip \
        7zip \
        xz-utils \
        tar \
        less \
        vim \
        nano \
        screen \
        procps \
        htop \
        btop \
        fastfetch \
        gdu \
        gping \
        tree \
        ripgrep \
        file \
        zstd \
        locales \
        golang-go \
    && rm -rf /var/lib/apt/lists/*

# Locale (UTF-8)
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && locale-gen
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# Go workspace (bin masuk ke PATH)
ENV GOPATH=/home/${USERNAME}/go \
    PATH=/home/${USERNAME}/go/bin:$PATH

# ---------------------------------------------------------------------------
# Non-root user with passwordless sudo
# ---------------------------------------------------------------------------
RUN groupadd --gid ${USER_GID} ${USERNAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
    && echo "${USERNAME} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

# ---------------------------------------------------------------------------
# Runtime
# ---------------------------------------------------------------------------
USER ${USERNAME}

# ---------------------------------------------------------------------------
# nvm + Node.js 24
# ---------------------------------------------------------------------------
ENV NVM_DIR=/home/${USERNAME}/.nvm

RUN bash -c 'set -e; \
        curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.41.0/install.sh | bash; \
        . "$NVM_DIR/nvm.sh"; \
        nvm install 24; \
        nvm alias default 24; \
        ln -sfn "$NVM_DIR/versions/node/$(nvm current)" "$NVM_DIR/current"'

ENV PATH=${NVM_DIR}/current/bin:$PATH

# ---------------------------------------------------------------------------
# CLI coding agents
# ---------------------------------------------------------------------------
# opencode
RUN curl -fsSL https://opencode.ai/v2/install | bash
ENV PATH=/home/${USERNAME}/.opencode/bin:$PATH

# Claude Code
RUN curl -fsSL https://claude.ai/install.sh | bash
ENV PATH=/home/${USERNAME}/.local/bin:$PATH

# Command Code (npm global, butuh Node.js dari nvm)
RUN npm i -g command-code@latest

# ---------------------------------------------------------------------------
# Homebrew
# ---------------------------------------------------------------------------
ENV HOMEBREW_NO_ANALYTICS=1 \
    HOMEBREW_NO_AUTO_UPDATE=1

RUN NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
ENV PATH=/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:$PATH

# Login shell (/etc/profile) me-reset PATH, jadi daftarkan ulang via profile.d
RUN PROFILE_PATH="/home/${USERNAME}/.local/bin:/home/${USERNAME}/.opencode/bin:/home/${USERNAME}/.nvm/current/bin:/home/${USERNAME}/go/bin:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin" \
    && echo "export PATH=\"${PROFILE_PATH}:\$PATH\"" \
        | sudo tee /etc/profile.d/devcontainer-path.sh > /dev/null

# ---------------------------------------------------------------------------
# Docker CLI + Compose (Docker-outside-of-Docker)
# Daemon tetap di host; container hanya memakai /var/run/docker.sock
# ---------------------------------------------------------------------------
RUN sudo apt-get update \
    && sudo apt-get install -y --no-install-recommends \
        docker-cli \
        docker-compose \
        docker-buildx \
    && sudo rm -rf /var/lib/apt/lists/*

# Samakan GID grup docker dengan socket host saat container start
RUN printf '%s\n' \
        '#!/bin/sh' \
        'set -e' \
        'if [ -S /var/run/docker.sock ]; then' \
        '    gid=$(stat -c "%g" /var/run/docker.sock)' \
        '    getent group "$gid" >/dev/null 2>&1 || sudo groupadd --gid "$gid" docker' \
        '    sudo usermod -aG "$gid" '"${USERNAME}" \
        'fi' \
        'exec "$@"' \
        | sudo tee /usr/local/share/docker-init.sh > /dev/null \
    && sudo chmod +x /usr/local/share/docker-init.sh

WORKDIR /workspaces

ENTRYPOINT ["/usr/local/share/docker-init.sh"]

# Keep the container alive; VS Code attaches to it.
CMD ["sleep", "infinity"]
