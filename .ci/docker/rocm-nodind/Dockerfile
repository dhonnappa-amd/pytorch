# syntax=docker/dockerfile:1
#
# No-dind ARC runner: GitHub Actions runner + ROCm test deps.
# Publish: rocm/framework-automation:pytorch-nodind-rocm7.14
#
# Built weekly on ubuntu-latest (.github/workflows/rocm-nodind-image.yml).
# No GPU and no corporate network. Public pulls only.

FROM ghcr.io/nod-ai/ossci-gitops/ghascale-rocm-dev:main

USER root
SHELL ["/bin/bash", "-euxo", "pipefail", "-c"]

ARG ROCM_VERSION=7.14
ARG THEROCK_INDEX_URL=https://repo.amd.com/rocm/whl-multi-arch/
ARG PYTORCH_REF=main

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    PIP_NO_CACHE_DIR=1 \
    PYTORCH_TEST_WITH_ROCM=1 \
    PYTORCH_RUNNER_ROCM_TAG=therock-${ROCM_VERSION}

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates \
      bc \
      build-essential \
      cmake \
      gdb \
      jq \
      kmod \
      libasound2t64 \
      libc++1 \
      libc++abi1 \
      libffi-dev \
      libgl1 \
      libjemalloc2 \
      libnuma-dev \
      libsndfile1 \
      libssl-dev \
      libtbb12 \
      python3 \
      python3-dev \
      python3-pip \
      python3-setuptools \
      python3-wheel \
      python3-venv \
      unzip \
      zip \
    && rm -rf /var/lib/apt/lists/*

ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv "${VIRTUAL_ENV}"
ENV PATH="${VIRTUAL_ENV}/bin:${PATH}"
RUN python -m pip install --upgrade pip setuptools wheel

RUN rocm_spec="${ROCM_VERSION}"; \
    if [[ "${ROCM_VERSION}" =~ ^[0-9]+\.[0-9]+$ ]]; then \
      rocm_spec="${ROCM_VERSION}.*"; \
    fi; \
    pip install --index-url "${THEROCK_INDEX_URL}" \
      "rocm[libraries,devel,device-all]==${rocm_spec}"

RUN ROCM_HOME="$(rocm-sdk path --root)"; \
    ROCM_BIN="$(rocm-sdk path --bin)"; \
    { \
      echo "export ROCM_PATH=${ROCM_HOME}"; \
      echo "export ROCM_HOME=${ROCM_HOME}"; \
      echo "export CMAKE_PREFIX_PATH=${ROCM_HOME}:\${CMAKE_PREFIX_PATH:-}"; \
      echo "export LD_LIBRARY_PATH=${ROCM_HOME}/lib:\${LD_LIBRARY_PATH:-}"; \
      echo "export PATH=${ROCM_BIN}:\${PATH}"; \
      echo "export PATH=${VIRTUAL_ENV}/bin:\${PATH}"; \
    } > /etc/rocm_env.sh; \
    echo "source /etc/rocm_env.sh" >> /etc/bash.bashrc

# Triton, origami, requirements-ci, and the torch wheel are installed at job time.
RUN chown -R runner:runner "${VIRTUAL_ENV}"

USER runner
WORKDIR /home/runner
