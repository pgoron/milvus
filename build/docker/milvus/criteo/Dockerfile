FROM filer-docker-registry-build.prod.crto.in/criteo-base-8-python:latest

ARG TARGETARCH
ARG MILVUS_ASAN_LIB

RUN dnf install -y libaio openblas

# Add Tini
RUN curl -L -o /tini https://github.com/krallin/tini/releases/download/v0.19.0/tini-$TARGETARCH && \
    chmod +x /tini

RUN mkdir -p /milvus/bin
COPY --chown=root:root --chmod=774 ./bin/milvus /milvus/bin/milvus

COPY --chown=root:root --chmod=774 ./configs/ /milvus/configs/

COPY --chown=root:root --chmod=774 ./lib/ /milvus/lib/

ENV PATH=/milvus/bin:$PATH
ENV LD_LIBRARY_PATH=/milvus/lib:$LD_LIBRARY_PATH:/usr/lib
ENV LD_PRELOAD=${MILVUS_ASAN_LIB}:/milvus/lib/libjemalloc.so
ENV MALLOC_CONF=background_thread:true

ENTRYPOINT ["/tini", "--"]

WORKDIR /milvus/
