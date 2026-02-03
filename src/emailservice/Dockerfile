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

# Enable unbuffered logging
ENV PYTHONUNBUFFERED=1

RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends \
    wget

# Download the grpc health probe
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.7 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

WORKDIR /email_server

# Add the application
COPY . .

RUN apt-get install -qq -y build-essential
RUN apt-get install -qq  -y libffi-dev
RUN apt-get install -qq  -y git
RUN pip install sealights-python-agent
RUN BUILD_NAME=$(date +%F_%T) && sl-python config --appname "otel_emailservice" --branchname master --buildname "${BUILD_NAME}" --exclude "*venv*" --scm none
RUN sl-python scan

#ENV PYTHONPATH=/usr/local/lib/python3.9/site-packages/python_agent/bootstrap/:$PYTHONPATH 
#ENV IS_MAIN_PROCESS=1

RUN mkdir -p /opt/python-startup
RUN cat > /opt/python-startup/sitecustomize.py <<'PY'
import python_agent.bootstrap.sitecustomize
PY
ENV PYTHONPATH="/opt/python-startup:${PYTHONPATH}"

ENV SL_TOKENFILE=/email_server/sltoken.txt
ENV SL_BUILDSESSIONIDFILE=/email_server/buildSessionId.txt
ENV SL_LABID=test

EXPOSE 8080
ENTRYPOINT ["python", "email_server.py" ]
