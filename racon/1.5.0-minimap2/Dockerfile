ARG RACON_VER="1.5.0"
ARG MINIMAP2_VER="2.28"

FROM ubuntu:jammy AS builder

ARG RACON_VER
ARG MINIMAP2_VER

# Install build dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
  perl \
  default-jre \
  gnuplot \
  libgomp1 \
  maven \
  git \
  wget \
  python3 \
  build-essential \
  cmake \
  zlib1g-dev \
  curl \
  bzip2 \
  libstdc++6 && \
  apt-get autoclean && rm -rf /var/lib/apt/lists/*

# Compile Racon with compatibility flags
RUN wget https://github.com/lbcb-sci/racon/archive/refs/tags/${RACON_VER}.tar.gz && \
  tar -xvf ${RACON_VER}.tar.gz && \
  cd racon-${RACON_VER} && \
  mkdir build && \
  cd build && \
  cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-march=x86-64 -mtune=generic" .. && \
  make && \
  cd ../../ && rm -rf ${RACON_VER}.tar.gz

# Add Racon to PATH
ENV PATH="/racon-${RACON_VER}/build/bin:${PATH}"

# Install Minimap2
RUN curl -L https://github.com/lh3/minimap2/releases/download/v${MINIMAP2_VER}/minimap2-${MINIMAP2_VER}_x64-linux.tar.bz2 | \
    tar -jxvf - --no-same-owner && \
    mv minimap2-${MINIMAP2_VER}_x64-linux /usr/local/minimap2 && \
    rm -rf minimap2-${MINIMAP2_VER}_x64-linux.tar.bz2

# Add Minimap2 to PATH
ENV PATH="/usr/local/minimap2:${PATH}"

FROM ubuntu:jammy AS app

ARG RACON_VER
ARG MINIMAP2_VER

# Metadata
LABEL base.image="ubuntu:jammy"
LABEL software="Racon + Minimap2"
LABEL description="Racon for assembly polishing and Minimap2 for alignment"

# Install runtime dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
  procps \
  wget \
  python3-minimal && \
  apt-get autoclean && rm -rf /var/lib/apt/lists/*

# Copy compiled binaries
COPY --from=builder /racon-${RACON_VER}/build/bin/* /usr/local/bin/
COPY --from=builder /usr/local/minimap2/* /usr/local/bin/

WORKDIR /data

# Locale settings
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8 PATH=${PATH}

CMD ["racon", "--help"]
