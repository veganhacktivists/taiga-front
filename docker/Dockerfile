# This source code is licensed under the terms of the
# GNU Affero General Public License found in the LICENSE file in
# the root directory of this source tree.
#
# Copyright (c) 2021-present Kaleidos INC

FROM nginx:1.23-alpine
LABEL maintainer="support@taiga.io"

# Install Node.js, npm, and other required packages
RUN set -eux; \
    apk update; \
    apk add --no-cache \
       bash \
       nodejs \
       npm;

# Copy necessary files for configuration
COPY docker/default.conf /etc/nginx/conf.d/default.conf
COPY docker/conf.json.template /
COPY docker/config_env_subst.sh /docker-entrypoint.d/30_config_env_subst.sh

# Copy your application source code (assuming it's in the current directory)
COPY . /app

RUN ls -la /app/docker

# Set working directory
WORKDIR /app

# Install dependencies and build the project
RUN npm install \
    && npx gulp deploy

# Move the built files to the appropriate location
RUN mv /app/tmp/* /usr/share/nginx/html/

# Clean up unnecessary files
RUN chmod +x /docker-entrypoint.d/30_config_env_subst.sh; \
    apk del --no-cache --purge; \
    rm -rf /var/cache/apk/* /tmp/* /var/tmp/* /app

# Expose necessary ports (if needed)
EXPOSE 80

# Default command
CMD ["nginx", "-g", "daemon off;"]
