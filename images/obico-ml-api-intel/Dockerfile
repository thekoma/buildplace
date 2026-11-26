# Obico's ML API (failure detection) for Intel GPUs: ONNX Runtime with the
# OpenVINO execution provider.
#
# Upstream defines this image (ml_api/Dockerfile.intel) and builds it on every
# push to `release`, but its workflow cannot publish it: GHCR rejects the push
# with "installation not allowed to Create organization package", so no Intel
# image exists anywhere. This rebuilds the same steps from the same commit. No
# source of ours.
#
# The stages below copy upstream's Dockerfile.intel at COMMIT step by step.
# Renovate moves COMMIT by itself, so the `src` stage checks that file's hash.
# When upstream changes the file, the build fails there instead of drifting
# silently: copy the new steps here and update DOCKERFILE_SHA256.

# renovate: datasource=git-refs depName=https://github.com/TheSpaghettiDetective/obico-server currentValue=release
ARG COMMIT=da0a2fe268b2e5e29711803d79dbe75765f243f1
ARG DOCKERFILE_SHA256=111c708a9a6ee87223a88f09054c59cdaabaceb4038fd72bbf05277decc9d03d

FROM python:3.12-slim AS src
ARG COMMIT
ARG DOCKERFILE_SHA256
# BuildKit fetches the exact commit itself, and only ml_api/ is copied.
ADD https://github.com/TheSpaghettiDetective/obico-server.git#${COMMIT}:ml_api /ml_api
RUN echo "${DOCKERFILE_SHA256}  /ml_api/Dockerfile.intel" | sha256sum -c - \
 || { echo "upstream ml_api/Dockerfile.intel changed at ${COMMIT}: copy its steps here" >&2; exit 1; }

# From here on: upstream's Dockerfile.intel, with `COPY <path>` from the build
# context turned into `COPY --from=src /ml_api/<path>`.
FROM python:3.12-slim

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install --no-install-recommends --assume-yes \
        ca-certificates curl gnupg patchelf wget \
        libgomp1 libglib2.0-0 ocl-icd-libopencl1 \
    && curl -fsSL https://repositories.intel.com/gpu/intel-graphics.key \
        | gpg --dearmor --output /usr/share/keyrings/intel-graphics.gpg \
    && echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/gpu/ubuntu jammy client' \
        > /etc/apt/sources.list.d/intel-gpu.list \
    && apt-get update \
    && apt-get install --no-install-recommends --assume-yes \
        intel-opencl-icd intel-level-zero-gpu \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=src /ml_api/requirements.txt /app/
RUN python3 -m pip install --no-cache-dir --upgrade pip \
    && python3 -m pip install --no-cache-dir \
        onnxruntime-openvino==1.24.1 \
        opencv-python-headless==4.12.0.88 \
    && find /usr/local/lib/python3.12/site-packages/onnxruntime \
        -type f -name '*.so*' -exec patchelf --clear-execstack {} + \
    && python3 -m pip install --no-cache-dir --requirement requirements.txt

COPY --from=src /ml_api/model/*.url /ml_api/model/*.sha256 /app/model/
COPY --from=src /ml_api/scripts/fetch_verified.sh /usr/local/bin/fetch_verified
RUN mkdir -p /model_cache/ml_api/onnx \
    && fetch_verified "$(tr -d '\r' < model/model-weights.onnx.url)" \
        /model_cache/ml_api/onnx/model-weights.onnx 50000000 \
        "$(tr -d '\r' < model/model-weights.onnx.sha256)"

COPY --from=src /ml_api/ /app/

EXPOSE 3333

CMD ["gunicorn", "--bind", "0.0.0.0:3333", "--workers", "1", "--timeout", "120", "wsgi"]
