# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

FROM almalinux:8
LABEL maintainer="SC Clouds <contato@scclouds.com.br> (@scclouds)"

RUN \
    yum clean metadata && \
    yum install -y epel-release && \
    yum clean metadata && \
    yum install -y \
        make \
        which \
        mlocate \
        tar \
        rpm-build \
        yum-utils \
        createrepo \
        mkisofs \
        git \
        java-11-openjdk-devel \
        tomcat \
        python2 python3 \
        gcc \
        gcc-c++ \
        jq \
        wget \
        bzip2 && \
    alternatives --set python /usr/bin/python2 && \
    python3 -m pip install --upgrade pip && \
    dnf install -y maven && \
    curl -sL https://rpm.nodesource.com/setup_14.x | bash - && \
    yum install -y nodejs && \
    yum clean all && rm -rf /var/cache/yum && \
    npm install -g vue@^2.x && \
    npm install -g vue-template-compiler@^2.x && \
    sed -i "s/%dist .el8$/%dist .el8/g" /etc/rpm/macros.dist


ENV JAVA_HOME=/usr/lib/jvm/java-11
RUN alternatives --set java $(readlink -f ${JAVA_HOME}/bin/java)

VOLUME /mnt/build
WORKDIR /mnt/build

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
COPY environment-info.sh /usr/local/bin/environment-info.sh

ENTRYPOINT ["docker-entrypoint.sh"]
