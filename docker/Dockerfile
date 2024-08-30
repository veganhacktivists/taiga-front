# This source code is licensed under the terms of the
# GNU Affero General Public License found in the LICENSE file in
# the root directory of this source tree.
#
# Copyright (c) 2021-present Kaleidos INC

FROM nginx:1.23
LABEL maintainer="support@taiga.io"

# Install Node.js, npm, and other required packages
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        git \
        bash \
        curl \
        optipng; \
    # Install the latest Node.js version
    curl -fsSL https://deb.nodesource.com/setup_current.x | bash -; \
    apt-get install -y nodejs; \
    # Clean up apt cache to reduce image size
    rm -rf /var/lib/apt/lists/*

# Copy necessary configuration files
COPY docker/default.conf /etc/nginx/conf.d/default.conf
COPY docker/conf.json.template /
COPY docker/config_env_subst.sh /docker-entrypoint.d/30_config_env_subst.sh

# Copy application source code including package.json and gulpfile.js
COPY package.json package-lock.json gulpfile.js /app/
COPY . /app/

# Set the working directory
WORKDIR /app

# Install dependencies and build the project
RUN npm install -g pnpm
RUN pnpm install
RUN pnpx gulp deploy

# Move the built files to the appropriate location
RUN mv /app/tmp/* /usr/share/nginx/html/

# Clean up unnecessary files
RUN chmod +x /docker-entrypoint.d/30_config_env_subst.sh; \
    apt-get remove -y --purge curl; \
    apt-get autoremove -y; \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /app

# Expose necessary ports (if needed)
EXPOSE 80

# Default command
CMD ["nginx", "-g", "daemon off;"]

