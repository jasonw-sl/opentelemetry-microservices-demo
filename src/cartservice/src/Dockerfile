# Copyright 2021 Google LLC
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

# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list
FROM mcr.microsoft.com/dotnet/sdk:6.0.200 as builder
WORKDIR /app
COPY cartservice.csproj .
RUN dotnet restore cartservice.csproj -r linux-x64
COPY . .
#RUN dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-x64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore
RUN dotnet publish cartservice.csproj -p:PublishSingleFile=false -r linux-x64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore

WORKDIR /sealights
COPY sltoken.txt .
ADD https://agents.sealights.co/dotnetcore/latest/sealights-dotnet-agent-linux-self-contained.tar.gz sealights-dotnet-agent-latest.tar.gz
RUN tar xvfz sealights-dotnet-agent-latest.tar.gz

#RUN BUILD_NAME=$(date +%F_%T) && dotnet SL.DotNet.dll config --appName otel_cartservice --branchName main --buildName ${BUILD_NAME} --buildSessionIdFile /sealights/buildSessionId.txt --includeNamespace "cartservice.*"
#RUN dotnet SL.DotNet.dll scan --buildSessionIdFile /sealights/buildSessionId.txt --binDir /app --ignoreGeneratedCode true

# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
#FROM mcr.microsoft.com/dotnet/sdk:6.0.400-alpine3.15-amd64
# debian
#FROM mcr.microsoft.com/dotnet/sdk:6.0.400-bullseye-slim-amd64
# ubuntu
FROM mcr.microsoft.com/dotnet/sdk:6.0.400-jammy-amd64
#FROM mcr.microsoft.com/dotnet/runtime-deps:6.0.3-alpine3.15-amd64
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.7 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe
WORKDIR /app
COPY --from=builder /cartservice .
COPY --from=builder /sealights /sealights
ENV ASPNETCORE_URLS http://*:7070

ENV COR_ENABLE_PROFILING=1
ENV COR_PROFILER="{3B1DAA64-89D4-4999-ABF4-6A979B650B7D}"
ENV COR_PROFILER_PATH_32=/sealights/libSL.DotNet.ProfilerLib.Linux.so
ENV COR_PROFILER_PATH_64=/sealights/libSL.DotNet.ProfilerLib.Linux.so
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER="{3B1DAA64-89D4-4999-ABF4-6A979B650B7D}"
ENV CORECLR_PROFILER_PATH_32=/sealights/libSL.DotNet.ProfilerLib.Linux.so
ENV CORECLR_PROFILER_PATH_64=/sealights/libSL.DotNet.ProfilerLib.Linux.so
#ENV SL_CollectorId=SL_CovCollectorAgentCartservice
ENV SL_PROFILER_INITIALIZECOLLECTOR=1
ENV SL_PROFILER_INITIALIZECOLLECTOR_MODE="cdAgent"
ENV SL_PROFILER_BLOCKING_CONNECTION_STARTUP="ASYNC"

ARG BUILD_NUMBER
ENV SL_SESSION_TOKENFILE=/sealights/sltoken.txt
ENV SL_GENERAL_APPNAME="otel_cartservice"
ENV SL_GENERAL_BRANCHNAME="main"
ENV SL_GENERAL_BUILDNAME=$BUILD_NUMBER
ENV SL_LABID="integ_test_otel"
#ENV SL_LABID="my_lab"
ENV SL_SCAN_BINDIR="/app"
ENV SL_SCAN_INCLUDENAMESPACES_0="cartservice.*"
ENV SL_SCAN_INCLUDEASSEMBLIES="*cartservice"
ENV SL_FEATURES_IDENTIFYMETHODSBYFQN="true"

#ENV SL_LogDir /tmp
#ENV SL_LogLevel=6
#ENV SL_Logging_toFile="true"
#ENV SL_LOGGING_ENABLED="true"
#ENV SL_LOGGING_FileName="/tmp/cdagent.log"

#RUN dotnet /sealights/SL.DotNet.dll startBackgroundTestListener --buildSessionIdFile /sealights/buildSessionId.txt --testListenerSessionKey SL_CovCollectorAgentCartservice &

#ENTRYPOINT ["dotnet","/sealights/SL.DotNet.dll","testListener","--buildSessionIdFile","/sealights/buildSessionId.txt","--labId","integ_test_otel","--workingDir","/app","--target","cartservice","--profilerLogDir","/tmp","--profilerLogLevel","6","--logAppendFile","true","--logFilename","/tmp/collector"]
ENTRYPOINT ["/app/cartservice"]
