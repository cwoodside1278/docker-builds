#Use ubuntu as base image
FROM ubuntu:noble

# metadata
LABEL base.image="ubuntu:noble"
LABEL version="2585414"
LABEL software="cdc-spn"
LABEL software.version="0.1"
LABEL description="Scripts and reference files to type streptococcus pneumoniae"
LABEL website="https://github.com/BenJamesMetcalf/Spn_Scripts_Reference"
LABEL license="https://github.com/BenJamesMetcalf/Spn_Scripts_Reference"
LABEL maintainer="Kelsey Florek"
LABEL maintainer="kelsey.florek@slh.wisc.edu"

# install dependencies
RUN apt-get update && apt-get install -y \
  perl \
  wget \
  git \
  ncbi-blast+ \
  bedtools \
  prodigal \
  python3 \
  python3-pip \
  python3-scipy \
  cutadapt \
  bowtie2 \
  zlib1g-dev \
  libbz2-dev \
  liblzma-dev \
  build-essential \
  libncurses5-dev \
  libcurl4-openssl-dev \
  autoconf \
  meson \
  ninja-build \
  pkg-config \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# install samtools 1.19
RUN wget https://github.com/samtools/samtools/releases/download/1.19/samtools-1.19.tar.bz2 \
  && tar -xjf samtools-1.19.tar.bz2 \
  && rm samtools-1.19.tar.bz2 \
  && cd samtools-1.19 \
  && ./configure --prefix=/usr/local \
  && make \
  && make install

# install srst2
RUN wget https://github.com/katholt/srst2/archive/v0.2.0.tar.gz \
  && tar -xzvf v0.2.0.tar.gz \
  && rm v0.2.0.tar.gz \
  && pip install --break-system-packages /srst2-0.2.0

# install freebayes v1.3.10
RUN git clone --recursive https://github.com/freebayes/freebayes.git \
  && cd freebayes \
  && git checkout v1.3.10 \
  && git submodule update --init --recursive \
  && meson setup build/ --buildtype release \
  && cd build \
  && ninja \
  && ninja install
  
# add the spn scripts pinned to commit 2585414
RUN git clone https://github.com/BenJamesMetcalf/Spn_Scripts_Reference.git \
  && cd Spn_Scripts_Reference \
  && git checkout 2585414

ENV PATH="/Spn_Scripts_Reference:/usr/local/bin:${PATH}"
