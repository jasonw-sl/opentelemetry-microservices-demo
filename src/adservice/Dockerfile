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

FROM openjdk:17-slim as builder

WORKDIR /app

COPY ["build.gradle", "gradlew", "./"]
COPY gradle gradle
RUN chmod +x gradlew
RUN ./gradlew downloadRepos

COPY . .
#RUN java -jar sl-build-scanner.jar -gradle -configfile slgradle.json -workspacepath .
RUN chmod +x gradlew
RUN ./gradlew installDist

FROM openjdk:17-slim

RUN apt-get -y update && apt-get install -qqy \
    wget \
    && rm -rf /var/lib/apt/lists/*

RUN GRPC_HEALTH_PROBE_VERSION=v0.4.7 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

WORKDIR /app
COPY --from=builder /app .
COPY agent/opentelemetry-javaagent.jar .

ENV JAVA_TOOL_OPTIONS="-javaagent:sl-cd-agent.jar -Dsl.log.toConsole=true -Dsl.log.level=debug -Dsl.tokenFile=sltoken.txt -Dsl.appName=otel_adservice -Dsl.labId=integ_test_otel -Dsl.includes=*hipstershop.AdService* -Dsl.excludes=*hipstershop.AdServiceGrpc* -Dsl.workspace=/app/build/install/hipstershop/lib/"
#ENV JAVA_TOOL_OPTIONS="-javaagent:sl-test-listener.jar -Dsl.tags=script,container -Dsl.labId=integ_test_otel -Dsl.log.toConsole=true -Dsl.log.level=info"


RUN apt-get update && apt-get install -y procps

EXPOSE 9555
ENTRYPOINT ["/app/build/install/hipstershop/bin/AdService"]
