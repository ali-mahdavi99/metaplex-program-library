FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive

ARG SOLANA_CLI
ARG ANCHOR_CLI=v0.0.0

ENV NODE_VERSION="v17.0.1"
ENV HOME="/root"
ENV PATH="${HOME}/.cargo/bin:${PATH}"
ENV PATH="${HOME}/.local/share/solana/install/active_release/bin:${PATH}"
ENV PATH="${HOME}/.nvm/versions/node/${NODE_VERSION}/bin:${PATH}"

ENV CLUSTER=devnet
ENV PROGRAM_NAME_TO_DEPLOY=
ENV SO_FILE_NAME=
ENV DEPLOYER_KEYPAIR=
ENV PROGRAM_KEYPAIR=
ENV USE_ANCHOR=False
ENV GIT_TAG_NAME=

# Install base utilities.
RUN mkdir -p /workdir && mkdir -p /tmp && \
    apt-get update -qq && apt-get upgrade -qq && apt-get install -qq \
    build-essential git curl wget jq pkg-config python3-pip \
    libssl-dev libudev-dev

# Install rust.
RUN curl "https://sh.rustup.rs" -sfo rustup.sh && \
    sh rustup.sh -y && \
    rustup component add rustfmt clippy

# Install node / npm / yarn.
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
ENV NVM_DIR="${HOME}/.nvm"
RUN . $NVM_DIR/nvm.sh && \
    nvm install ${NODE_VERSION} && \
    nvm use ${NODE_VERSION} && \
    nvm alias default node && \
    npm install -g yarn

# Install Solana tools.
RUN sh -c "$(curl -sSfL https://release.solana.com/${SOLANA_CLI}/install)"

# Install Anchor (if we need it)
RUN if [ "${ANCHOR_CLI}" != "v0.0.0" ]; then \
        export USE_ANCHOR=True && \
        cargo install --git https://github.com/project-serum/anchor --tag ${ANCHOR_CLI} anchor-cli --locked; \
    fi

WORKDIR /workdir

# Deploy
CMD git clone -b ${GIT_TAG_NAME} https://github.com/metaplex-foundation/metaplex-program-library && \
    cd metaplex-program-library/${PROGRAM_NAME_TO_DEPLOY}/program && \
    touch updateAuthorityKey.json && \
    if [ ${USE_ANCHOR} ]; then anchor build; else cargo build-bpf --bpf-out-dir ../../target/deploy/; fi && \
    echo ${DEPLOYER_KEYPAIR} > deployer.json && \
    echo ${PROGRAM_KEYPAIR} > program_id.json && \
    if [ ${USE_ANCHOR} ]; then solana program deploy ./target/deploy/${SO_FILE_NAME} -u ${CLUSTER} -k deployer.json; else solana program deploy ../../target/deploy/${SO_FILE_NAME} -u ${CLUSTER} -k deployer.json; fi