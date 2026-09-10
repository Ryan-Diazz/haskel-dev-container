# ============================================================
# Build: direnv
# ============================================================
ARG GOLANG_VERSION=1.26.3

FROM golang:${GOLANG_VERSION}-bookworm AS direnv-builder

ARG DIRENV_VERSION=2.37.1

RUN go install github.com/direnv/direnv/v2@v${DIRENV_VERSION}

# ============================================================
# Build: Haskell environment
# ============================================================
FROM debian:bookworm-slim AS haskell-builder

ARG GHC_VERSION=9.10.3

ARG CABAL_VERSION=3.12.1.0

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gcc \
    g++ \
    git \
    libc6-dev \
    libffi-dev \
    libgmp-dev \
    libncurses-dev \
    libtinfo-dev \
    make \
    pkg-config \
    xz-utils \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# Create the same user/path that will exist in the final image.
RUN useradd \
    --create-home \
    --shell /bin/bash \
    --uid 1000 \
    vscode

USER vscode
WORKDIR /home/vscode

ENV PATH="/home/vscode/.cabal/bin:/home/vscode/.local/bin:/home/vscode/.ghcup/bin:${PATH}"

# GHCup
RUN curl --proto '=https' \
    --tlsv1.2 \
    -sSf \
    https://get-ghcup.haskell.org | \
    BOOTSTRAP_HASKELL_NONINTERACTIVE=1 \
    BOOTSTRAP_HASKELL_GHC_VERSION=${GHC_VERSION} \
    BOOTSTRAP_HASKELL_CABAL_VERSION=${CABAL_VERSION} \
    BOOTSTRAP_HASKELL_INSTALL_STACK=1 \
    BOOTSTRAP_HASKELL_INSTALL_HLS=1 \
    sh

RUN ghcup set ghc ${GHC_VERSION}

RUN mkdir -p /home/vscode/.local && \
    mkdir -p /home/vscode/.cabal && \
    mkdir -p /home/vscode/.ghcup


# Cabal configuration
RUN mkdir -p /home/vscode/.cabal && \
    printf '%s\n' \
        'jobs: $ncpus' \
        'documentation: False' \
        > /home/vscode/.cabal/config


# ============================================================
# Final: Haskell devcontainer
# ============================================================
FROM debian:bookworm-slim

ARG GHC_VERSION=9.10.3
ARG CABAL_VERSION=3.12.1.0

ENV DEBIAN_FRONTEND=noninteractive

# Only runtime dependencies required by the Haskell binaries.
# Only runtime dependencies required by the Haskell binaries.
RUN apt-get update && apt-get install -y --no-install-recommends \
    bash \
    ca-certificates \
    gcc \
    g++ \
    git \
    libffi8 \
    libffi-dev \
    libgmp10 \
    libgmp-dev \
    libncurses6 \
    libtinfo6 \
    zlib1g \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# Create vscode user.
RUN useradd \
    --create-home \
    --shell /bin/bash \
    --uid 1000 \
    vscode

# --------------------
# direnv
# --------------------
COPY --from=direnv-builder /go/bin/direnv /usr/local/bin/direnv

# --------------------
# Haskell toolchain
# --------------------
COPY --from=haskell-builder \
    /home/vscode/.ghcup \
    /home/vscode/.ghcup

COPY --from=haskell-builder \
    /home/vscode/.cabal \
    /home/vscode/.cabal

COPY --from=haskell-builder \
    /home/vscode/.local \
    /home/vscode/.local

# Make sure ownership is correct.
RUN chown -R vscode:vscode \
    /home/vscode/.ghcup \
    /home/vscode/.cabal \
    /home/vscode/.local

USER vscode

ENV PATH="/home/vscode/.cabal/bin:/home/vscode/.local/bin:/home/vscode/.ghcup/bin:${PATH}"

# --------------------
# Shell configuration
# --------------------
RUN printf '%s\n' \
    'eval "$(direnv hook bash)"' \
    'cat ~/.welcome_message 2>/dev/null || true' \
    >> /home/vscode/.bashrc

RUN printf '%s\n' \
    'eval "$(direnv hook zsh)"' \
    >> /home/vscode/.zshrc

RUN printf '%s\n' \
    "==============================" \
    "Haskell Dev Environment Ready!" \
    "==============================" \
    "- GHC ${GHC_VERSION}" \
    "- Cabal ${CABAL_VERSION}" \
    "Tools: Hoogle, Ormolu, HLS, ghci" \
    > /home/vscode/.welcome_message

WORKDIR /workspaces/haskell

ENV DEBIAN_FRONTEND=dialog