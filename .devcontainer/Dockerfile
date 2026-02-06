FROM node:20

ARG TZ
ENV TZ="$TZ"

# Install basic development tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    less \
    git \
    procps \
    sudo \
    ca-certificates \
    fzf \
    zsh \
    man-db \
    unzip \
    gnupg2 \
    gh \
    jq \
    nano \
    vim \
    && apt-get clean && rm -rf /var/lib/apt/lists/*


# --- Dyalog APL (v20) ---
ARG DYALOG_APL_VERSION=20.0.52753
RUN set -eux; \
    arch="$(dpkg --print-architecture)"; \
    case "$arch" in \
    arm64) dyalog_deb="linux_64_${DYALOG_APL_VERSION}_unicode.aarch64.deb" ;; \
    amd64) dyalog_deb="linux_64_${DYALOG_APL_VERSION}_unicode.x86_64.deb" ;; \
    *) echo "Unsupported architecture: $arch" >&2; exit 1 ;; \
    esac; \
    url="https://www.dyalog.com/uploads/php/download.dyalog.com/download.php?file=20.0%2F${dyalog_deb}"; \
    wget -O "/tmp/${dyalog_deb}" "$url"; \
    dpkg -i "/tmp/${dyalog_deb}" || (apt-get update && apt-get install -y -f --no-install-recommends); \
    rm -f "/tmp/${dyalog_deb}"; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# Install .NET SDK
RUN wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh \
    && chmod +x dotnet-install.sh \
    && ./dotnet-install.sh --channel 8.0 --install-dir /usr/local/dotnet \
    && rm dotnet-install.sh

ENV DOTNET_ROOT=/usr/local/dotnet
ENV PATH=$PATH:/usr/local/dotnet

# Ensure default node user has access to npm global install path
RUN mkdir -p /usr/local/share/npm-global \
    && chown -R node:node /usr/local/share/npm-global

ARG USERNAME=node

# Persist shell history
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
    && mkdir /commandhistory \
    && touch /commandhistory/.bash_history \
    && chown -R "$USERNAME":"$USERNAME" /commandhistory

ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/node/.claude \
    && chown -R node:node /workspace /home/node/.claude

WORKDIR /workspace

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
    wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
    sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
    rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Set up non-root user
USER node

ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin:/home/node/.local/bin
ENV SHELL=/bin/zsh
ENV EDITOR=nano
ENV VISUAL=nano

ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
    -p git \
    -p fzf \
    -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
    -a "source /usr/share/doc/fzf/examples/completion.zsh" \
    -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
    -x

# Install Claude Code (native install - auto-updates)
RUN curl -fsSL https://claude.ai/install.sh -o /tmp/claude-install.sh \
    && bash /tmp/claude-install.sh \
    && rm -f /tmp/claude-install.sh \
    && command -v claude

# Give node user sudo access
USER root
RUN echo "node ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/node \
    && chmod 0440 /etc/sudoers.d/node
USER node
