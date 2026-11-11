# Debian 13 (trixie)
FROM debian:13

ARG DEBIAN_FRONTEND=noninteractive
ARG USERNAME=vscode
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
        xz-utils \
        tar \
        less \
        vim \
        nano \
        procps \
        htop \
        tree \
        ripgrep \
        locales \
    && rm -rf /var/lib/apt/lists/*

# Locale (UTF-8)
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && locale-gen
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

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
WORKDIR /workspaces

# Keep the container alive; VS Code attaches to it.
CMD ["sleep", "infinity"]
