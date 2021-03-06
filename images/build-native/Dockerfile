#
# Copyright (c) 2019, 2020 Oracle and/or its affiliates. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM debian:buster-slim as build

RUN set -x \
    && apt-get -y update \
    && apt-get -y install gcc g++ git make python zlib1g-dev wget \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build

RUN set -x \
    && git clone --depth 1 https://github.com/graalvm/mx.git \
    && wget -q --output-document jvmcijdk.tar.gz https://github.com/graalvm/openjdk8-jvmci-builder/releases/download/jvmci-20.1-b02/openjdk-8u252+09-jvmci-20.1-b02-linux-amd64.tar.gz \
    && mkdir jvmcijdk8 \
    && tar -xz --strip 1 -f jvmcijdk.tar.gz -C /build/jvmcijdk8 \
    && rm jvmcijdk.tar.gz

RUN git clone --depth 1 --single-branch https://github.com/oracle/graal.git --branch release/graal-vm/20.1
WORKDIR /build/graal/vm
RUN export JAVA_HOME=/build/jvmcijdk8 \
    && /build/mx/mx --dy /substratevm --force-bash-launchers=true --disable-polyglot --skip-libraries=true build

WORKDIR /build/graal/vm/latest_graalvm
RUN LONG_NAME=$(ls) \
    && SHORT_NAME=graalvm \
    && mv $LONG_NAME $SHORT_NAME

FROM debian:stretch-slim as final

RUN set -x \
    && apt-get -y update \
    && apt-get -y install gcc zlib1g-dev

COPY --from=build /build/graal/vm/latest_graalvm/graalvm /usr/local/graalvm
COPY src/main/c/libfnunixsocket.so /function/runtime/lib/


ENV GRAALVM_HOME=/usr/local/graalvm
WORKDIR /function
