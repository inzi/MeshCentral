FROM --platform=$BUILDPLATFORM node:22-alpine AS builder

RUN mkdir -p /opt/meshcentral/meshcentral
COPY ./ /opt/meshcentral/meshcentral/
WORKDIR /opt/meshcentral

ARG DISABLE_MINIFY=""
ARG DISABLE_TRANSLATE=""


RUN if ! [ -z "$DISABLE_MINIFY" ] && [ "$DISABLE_MINIFY" != "yes" ] && [ "$DISABLE_MINIFY" != "YES" ] \
    && [ "$DISABLE_MINIFY" != "true" ] && [ "$DISABLE_MINIFY" != "TRUE" ]; then \
    echo -e "\e[0;31;49mInvalid value for build argument DISABLE_MINIFY, possible values: yes/true\e[;0m"; exit 1; \
    fi
RUN if ! [ -z "$DISABLE_TRANSLATE" ] && [ "$DISABLE_TRANSLATE" != "yes" ] && [ "$DISABLE_TRANSLATE" != "YES" ] \
    && [ "$DISABLE_TRANSLATE" != "true" ] && [ "$DISABLE_TRANSLATE" != "TRUE" ]; then \
    echo -e "\e[0;31;49mInvalid value for build argument DISABLE_TRANSLATE, possible values: yes/true\e[;0m"; exit 1; \
    fi

# install translate/minify modules if need too
RUN if [ -z "$DISABLE_MINIFY" ] || [ -z "$DISABLE_TRANSLATE" ]; then cd meshcentral && npm install html-minifier@4.0.0 jsdom@22.1.0 esprima@4.0.1; fi

# first extractall if need too
RUN if [ -z "$DISABLE_MINIFY" ] || [ -z "$DISABLE_TRANSLATE" ]; then cd meshcentral/translate && node translate.js extractall; fi

# minify files
RUN if [ -z "$DISABLE_MINIFY" ]; then cd meshcentral/translate && node translate.js minifyall; fi

# translate
RUN if [ -z "$DISABLE_TRANSLATE" ]; then cd meshcentral/translate && node translate.js translateall; fi

# cleanup
RUN if [ -z "$DISABLE_MINIFY" ] || [ -z "$DISABLE_TRANSLATE" ]; then cd meshcentral && npm remove html-minifier jsdom esprima; fi
RUN rm -rf /opt/meshcentral/meshcentral/docker
RUN rm -rf /opt/meshcentral/meshcentral/node_modules


FROM --platform=$TARGETPLATFORM alpine:3.21

#Add non-root user, add installation directories and assign proper permissions
RUN mkdir -p /opt/meshcentral/meshcentral

# meshcentral installation
WORKDIR /opt/meshcentral

RUN apk update \
    && apk add --no-cache --update tzdata nodejs npm bash python3 make gcc g++ \
    && rm -rf /var/cache/apk/*
RUN npm install -g npm@latest

ARG INCLUDE_MONGODBTOOLS=""
ARG PREINSTALL_LIBS="false"

# environment variables
ENV NODE_ENV="production"
ENV CONFIG_FILE="config.json"

# environment variables for initial configuration file
ENV USE_MONGODB="false"
ENV MONGO_INITDB_ROOT_USERNAME="root"
ENV MONGO_INITDB_ROOT_PASSWORD="pass"
ENV MONGO_URL=""
ENV HOSTNAME="localhost"
ENV ALLOW_NEW_ACCOUNTS="true"
ENV ALLOWPLUGINS="false"
ENV LOCALSESSIONRECORDING="true"
ENV MINIFY="false"
ENV WEBRTC="false"
ENV IFRAME="false"
ENV SESSION_KEY=""
ENV REVERSE_PROXY="false"
ENV REVERSE_PROXY_TLS_PORT=""
ENV ARGS=""
ENV ALLOWED_ORIGIN="false"

RUN if ! [ -z "$INCLUDE_MONGODBTOOLS" ] && [ "$INCLUDE_MONGODBTOOLS" != "yes" ] && [ "$INCLUDE_MONGODBTOOLS" != "YES" ] \
    && [ "$INCLUDE_MONGODBTOOLS" != "true" ] && [ "$INCLUDE_MONGODBTOOLS" != "TRUE" ]; then \
    echo -e "\e[0;31;49mInvalid value for build argument INCLUDE_MONGODBTOOLS, possible values: yes/true\e[;0m"; exit 1; \
    fi

RUN if ! [ -z "$INCLUDE_MONGODBTOOLS" ]; then apk add --no-cache mongodb-tools; fi

# copy files from builder-image
COPY --from=builder /opt/meshcentral/meshcentral /opt/meshcentral/meshcentral
COPY ./docker/startup.sh ./startup.sh
COPY ./docker/config.json.template /opt/meshcentral/config.json.template

# install dependencies from package.json
RUN cd meshcentral && npm install

# NOTE: ALL MODULES MUST HAVE A VERSION NUMBER AND THE VERSION MUST MATCH THAT USED IN meshcentral.js mainStart()
RUN if ! [ -z "$INCLUDE_MONGODBTOOLS" ]; then cd meshcentral && npm install mongodb@4.17.2; fi
RUN if ! [ -z "$PREINSTALL_LIBS" ] && [ "$PREINSTALL_LIBS" == "true" ]; then cd meshcentral && npm install ssh2@1.16.0 semver@7.5.4 nodemailer@6.9.15 image-size@1.2.1 wildleek@2.0.0 otplib@12.0.1 yubikeyotp@0.2.0; fi

EXPOSE 80 443 4433

# volumes
VOLUME /opt/meshcentral/meshcentral-data
VOLUME /opt/meshcentral/meshcentral-files
VOLUME /opt/meshcentral/meshcentral-web
VOLUME /opt/meshcentral/meshcentral-backups

CMD ["bash", "/opt/meshcentral/startup.sh"]
