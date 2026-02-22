ARG CLAIR3_VER="2.0.0"

FROM mambaorg/micromamba:2.5.0-ubuntu24.04 AS builder

ARG CLAIR3_VER

USER root

WORKDIR /

RUN apt-get update && apt-get install -y \
    wget \
    bzip2 \
    make \
    g++ \
    libboost-graph-dev

RUN micromamba install --name base -c conda-forge -c bioconda \
    python=3.10.0 \
    tensorflow-cpu=2.11.0 \
    pytables \
    pigz \
    cffi \
    parallel=20191122 \
    zstd \
    samtools=1.15.1 \
    whatshap=1.7 \
    xz \
    zlib \
    bzip2 \
    automake \
    curl &&\
    micromamba clean -a -f -y

# Added to mitigate the PyPy download issue
RUN apt-get update && apt-get install -y pypy3 pypy3-dev python3-pip && \
    pypy3 -m pip install --break-system-packages mpmath==1.2.1

ENV PATH="$PATH:/opt/conda/bin/"

ARG MAMBA_DOCKERFILE_ACTIVATE=1

RUN pypy3 -m pip install --break-system-packages mpmath==1.2.1 &&\
    pip install tensorflow-addons

RUN wget -q https://github.com/HKU-BAL/Clair3/archive/refs/tags/v${CLAIR3_VER}.tar.gz &&\
    tar -xvf v${CLAIR3_VER}.tar.gz &&\
    cd Clair3-${CLAIR3_VER}/preprocess/realign/ &&\
    g++ -std=c++14 -O1 -shared -fPIC -o realigner ssw_cpp.cpp ssw.c realigner.cpp && \
    g++ -std=c++11 -shared -fPIC -o debruijn_graph -O3 debruijn_graph.cpp &&\
    cd ../.. &&\
    make &&\
    mkdir /clair3 &&\
    cp -rv clair3 preprocess postprocess scripts shared /clair3 &&\
    cp clair3.py run_clair3.py run_clair3.sh /clair3 &&\
    cp longphase libclair3* /clair3 &&\
    cp LICENSE.md /clair3

RUN mkdir /clair3/models &&\
    wget -q http://www.bio8.cs.hku.hk/clair3/clair3_models/clair3_models.tar.gz &&\    
    tar --no-same-owner -C /clair3/models -xvf clair3_models.tar.gz

## App ##
FROM mambaorg/micromamba:2.5.0-ubuntu24.04 AS app

ARG CLAIR3_VER

USER root

WORKDIR /

LABEL base.image="mambaorg/micromamba:2.5.0-ubuntu24.04"
LABEL dockerfile.version="1"
LABEL software="CLAIR3"
LABEL software.version="${CLAIR3_VER}"
LABEL description="Clair3 is a germline small variant caller for long-reads."
LABEL website="https://github.com/HKU-BAL/Clair3"
LABEL license="https://github.com/HKU-BAL/Clair3/blob/main/LICENSE.md"
LABEL maintainer="Kutluhan Incekara"
LABEL maintainer.email="kutluhan.incekara@ct.gov"

RUN apt-get update && apt-get install --no-install-recommends -y\
    procps && \
    apt-get autoclean && rm -rf /var/lib/apt/lists/*

RUN micromamba install --name base -c conda-forge -c bioconda \
    python=3.10.0 \
    numpy=1.24.3 \
    tensorflow-cpu=2.11.0 \
    pytables \
    pigz \
    cffi \
    parallel=20191122 \
    zstd \
    samtools=1.15.1 \
    whatshap=1.7 &&\
    micromamba clean -a -f -y &&\
    rm -rf /opt/conda/pkgs/

# Added to mitigate the PyPy download issue
RUN apt-get update && apt-get install -y pypy3 pypy3-dev python3-pip && \
    pypy3 -m pip install --break-system-packages mpmath==1.2.1

ENV PATH="/opt/conda/bin/:/clair3:${PATH}" \
    LC_ALL=C.UTF-8

RUN pypy3 -m pip install --break-system-packages mpmath==1.2.1 &&\
    pip install tensorflow-addons

COPY --from=builder /clair3 /clair3

CMD ["run_clair3.sh", "-h"]

WORKDIR /data

## Test ##
FROM app AS test

RUN run_clair3.sh --help || true

RUN apt-get update && apt-get install -y wget 

RUN wget -q https://raw.githubusercontent.com/StaPH-B/docker-builds/master/build-files/clair3/1.0.9/ont_quick_test.sh && \
    chmod +x ont_quick_test.sh &&\
    ./ont_quick_test.sh

# list models
RUN ls /clair3/models

# list tools installed with micromamba
RUN micromamba list
