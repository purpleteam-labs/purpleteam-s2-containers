# Copyright (C) 2017-2022 BinaryMist Limited. All rights reserved.

# Use of this software is governed by the Business Source License
# included in the file /licenses/bsl.md

# As of the Change Date specified in that file, in accordance with
# the Business Source License, use of this software will be governed
# by the Apache License, Version 2.0

# This file is only used in cloud env.

FROM owasp/zap2docker-stable:s2021-12-16

ARG PT_GROUP_ID

USER root
RUN addgroup --system --gid ${PT_GROUP_ID} purpleteam && adduser zap purpleteam
USER zap

