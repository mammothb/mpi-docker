ARG BASE_IMAGE=ubuntu:24.04

FROM ${BASE_IMAGE} as dev-base

ARG DEBIAN_FRONTEND=noninteractive
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        openssh-client \
        wget; \
    rm -rf /var/lib/apt/lists/* 
ENV PATH /opt/conda/bin:$PATH

FROM dev-base as conda
RUN set -eux; \
    cd /opt; \
    curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"; \
    bash ./Miniforge3-* -b -p /opt/conda -s; \
    /opt/conda/bin/conda install -y \
        gcc=13.2 \
        gxx=13.2 \
        openmpi=5.0.3 \
        sysroot_linux-64=2.17; \
    /opt/conda/bin/conda clean -ya; \
    rm -rf ./Miniforge3-*

FROM ${BASE_IMAGE} as official
ARG DEBIAN_FRONTEND=noninteractive
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        openssh-client \
        openssh-server; \
    rm -rf /var/lib/apt/lists/* 
# Configure SSH
RUN set -eux; \
    sed -i \
        's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' \
        /etc/ssh/sshd_config; \
    echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config; \
    echo "    UserKnownHostsFile /dev/null" >> /etc/ssh/ssh_config
COPY ssh /root/.ssh
RUN set -eux; \
    chmod 700 /root/.ssh; \
    chmod 600 /root/.ssh/id_ed25519; \
    chmod 644 /root/.ssh/id_ed25519.pub; \
    cp /root/.ssh/id_ed25519.pub /root/.ssh/authorized_keys 

COPY --from=conda /opt/conda /opt/conda
ENV PATH /opt/conda/bin:$PATH
RUN set -eux; \
    echo "export PATH=${PATH}" >> /root/.bashrc; \
    echo "alias mpirun='mpirun --allow-run-as-root'" >> /root/.bashrc

WORKDIR /workspace

COPY HelloWorld.cpp .

CMD ["service", "ssh", "start", "-D"]