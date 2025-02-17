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
        wget \
        unzip \
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

COPY . /taiga
WORKDIR /taiga

# Install dependencies and build the project
RUN npm install -g pnpm
RUN pnpm install
RUN which optipng

RUN optipng -version

RUN pnpx gulp deploy

WORKDIR /

RUN mv /conf.json.template taiga/dist/; \
    chmod +x /docker-entrypoint.d/30_config_env_subst.sh; \
    # Install taiga-front contribs
    mkdir /taiga/plugins; \
    cd /taiga/plugins; \
    # Slack
    wget https://github.com/taigaio/taiga-contrib-slack/archive/6.8.0.zip -O source.zip; \
    unzip -j source.zip "taiga-contrib-slack-6.8.0/front/dist/*" -d slack; \
    rm source.zip; \
    # Github
    wget http://github.com/taigaio/taiga-contrib-github-auth/archive/6.8.0.zip -O source.zip; \
    unzip -j source.zip "taiga-contrib-github-auth-6.8.0/front/dist/*" -d github-auth; \
    rm source.zip; \
    # Gitlab
    wget http://github.com/taigaio/taiga-contrib-gitlab-auth/archive/6.8.0.zip -O source.zip; \
    unzip -j source.zip "taiga-contrib-gitlab-auth-6.8.0/front/dist/*" -d gitlab-auth; \
    rm source.zip; \
    # Ready for nginx
    mv /taiga/dist/* /usr/share/nginx/html/; \
    mv /taiga/plugins /usr/share/nginx/html/; \
    apt-get remove -y --purge curl git wget unzip; \
    apt-get autoremove -y; \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /taiga
