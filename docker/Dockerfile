# Copyright (c) 2019 Cisco and/or its affiliates.
# 
# This software is licensed to you under the terms of the Cisco Sample
# Code License, Version 1.1 (the "License"). You may obtain a copy of the
# License at
#
#                https://developer.cisco.com/docs/licenses
#
# All use of the material herein must be in accordance with the terms of
# the License. All rights not expressly granted by the License are
# reserved. Unless required by applicable law or agreed to separately in
# writing, software distributed under the License is distributed on an "AS
# IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
# or implied.

FROM devhub-docker.cisco.com/iox-docker/ie4k/base-rootfs as build-env

RUN opkg update && \
    opkg install iox-toolchain && \
    opkg install libxml2-dev && \
    opkg install cppunit-dev && \
    opkg install cmake && \
    opkg install pkgconfig
COPY main.c /opt/src/
RUN cd /opt/src && \
    cc -o topproc main.c

FROM devhub-docker.cisco.com/iox-docker/ie4k/base-rootfs

COPY --from=build-env /opt/src/topproc /usr/local/bin/

CMD ["/usr/local/bin/topproc"]
