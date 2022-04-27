# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM python:3.9-slim

# get packages
COPY requirements.txt .
RUN pip install -r requirements.txt

# show python logs as they occur
ENV PYTHONUNBUFFERED=0

RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends \
    wget

# download the grpc health probe
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.7 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

WORKDIR /recommendationservice

# add files into working directory
COPY . .

RUN apt-get install -qq -y build-essential
RUN apt-get install -qq  -y libffi-dev
RUN apt-get install -qq  -y git
RUN pip install sealights-python-agent
RUN BUILD_NAME=$(date +%F_%T) && sl-python config --appname "otel_recommendationservice" --branchname master --buildname "${BUILD_NAME}" --exclude "*venv*" --scm none
RUN sl-python build
RUN apt-get update && apt-get install -y procps

EXPOSE 8080

ENV SL_DEBUG=true
ENV SL_LOG_LEVEL=debug
ENTRYPOINT ["opentelemetry-instrument", "sl-python", "run", "--labid", "integ_test_otel", "python", "recommendation_server.py"]
