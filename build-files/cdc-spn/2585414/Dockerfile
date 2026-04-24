# Use ubuntu as base image
FROM ubuntu:noble AS app

# metadata
LABEL base.image="ubuntu:noble"
LABEL version="2585414"
LABEL software="cdc-spn"
LABEL dockerfile.version="1"
LABEL software.version="0.1"
LABEL description="Scripts and reference files to type streptococcus pneumoniae"
LABEL website="https://github.com/BenJamesMetcalf/Spn_Scripts_Reference"
LABEL license="https://github.com/BenJamesMetcalf/Spn_Scripts_Reference"
LABEL maintainer="Kelsey Florek"
LABEL maintainer.email="kelsey.florek@slh.wisc.edu"

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

# install miniconda for srst2
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /miniconda.sh \
  && bash /miniconda.sh -b -p /opt/conda \
  && rm /miniconda.sh \
  && /opt/conda/bin/conda clean -a

ENV PATH="/opt/conda/bin:${PATH}"

# install srst2 via bioconda
RUN conda install -y -c bioconda -c conda-forge srst2 \
  && conda clean -a

# install freebayes v1.3.6
RUN git clone --recursive https://github.com/freebayes/freebayes.git \
  && cd freebayes \
  && git checkout v1.3.6 \
  && git submodule update --init --recursive \
  && sed -i '1s/^/#include <limits>\n/' intervaltree/IntervalTree.h \
  && meson setup build/ --buildtype release \
  && cd build \
  && ninja \
  && ninja install

# add the spn scripts pinned to commit 2585414
RUN git clone https://github.com/BenJamesMetcalf/Spn_Scripts_Reference.git \
  && cd Spn_Scripts_Reference \
  && git checkout 2585414

WORKDIR /data

#ENV PATH="/Spn_Scripts_Reference:/usr/local/bin:${PATH}"
ENV PATH="/opt/conda/bin:/Spn_Scripts_Reference:/usr/local/bin:${PATH}" \
  LC_ALL=C

  
# ---- test stage ----
FROM app AS test

WORKDIR /test_data

# install sra-toolkit for test data download
RUN apt-get update && apt-get install -y sra-toolkit \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# run test - ERR559060 is Streptococcus pneumoniae Serotype 1
RUN fasterq-dump ERR559060 --split-files -O /test_data \
  && perl /Spn_Scripts_Reference/SPN_Serotyper.pl \
     -1 /test_data/ERR559060_1.fastq \
     -2 /test_data/ERR559060_2.fastq \
     -r /Spn_Scripts_Reference \
     -o /test_output \
     -n ERR559060 \
  && echo "SPN_Serotyper.pl test passed" \
  && rm -rf /test_data /test_output
