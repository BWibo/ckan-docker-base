FROM alpine:3.14

ARG DATAPUSHER_VERSION=${DATAPUSHER_VERSION}

ENV TZ=UTC
ENV APP_DIR=/srv/app
ENV GIT_URL https://github.com/ckan/datapusher.git
ENV JOB_CONFIG ${APP_DIR}/datapusher_settings.py

WORKDIR ${APP_DIR}

# Set up timezone
RUN apk add --no-cache tzdata
RUN echo ${TZ} > /etc/timezone

RUN apk add --no-cache \
    python3 \
    py3-pip \
    py3-wheel \
    libffi-dev \
    libmagic \
    libressl-dev \
    libxslt \
    uwsgi \
    uwsgi-http \
    uwsgi-corerouter \
    uwsgi-python \
    # Temporary packages to build DataPusher requirements
    && apk add --no-cache --virtual .build-deps \
    gcc \
    git \
    musl-dev \
    python3-dev \
    libxml2-dev \
    libxslt-dev \
    openssl-dev \
    cargo \
    wget

RUN mkdir -p ${APP_DIR}/src && cd ${APP_DIR}/src && \
    git clone -b ${DATAPUSHER_VERSION} --depth=1 --single-branch ${GIT_URL} && \
    cd datapusher && \
    python3 setup.py install
    
RUN  cd ${APP_DIR}/src && \ 
     #pip3 install --no-cache-dir -r requirements.txt
     pip3 install --no-cache-dir -r  https://raw.githubusercontent.com/ckan/datapusher/${DATAPUSHER_VERSION}/requirements.txt

RUN apk del .build-deps && \
    cp ${APP_DIR}/src/datapusher/deployment/*.* ${APP_DIR} && \
    # Remove default values in ini file
    sed -i '/http/d' ${APP_DIR}/datapusher-uwsgi.ini && \
    sed -i '/wsgi-file/d' ${APP_DIR}/datapusher-uwsgi.ini && \
    sed -i '/virtualenv/d' ${APP_DIR}/datapusher-uwsgi.ini && \
    rm -rf ${APP_DIR}/src

# Create a local user and group to run the app
RUN addgroup -g 92 -S ckan && \
    adduser -u 92 -h /srv/app -H -D -S -G ckan ckan    

USER ckan

EXPOSE 8800
CMD ["sh", "-c", \
    "uwsgi --plugins=http,python --enable-threads --http=[::]:8800 --socket=/tmp/uwsgi.sock --ini=`echo ${APP_DIR}`/datapusher-uwsgi.ini --wsgi-file=`echo ${APP_DIR}`/datapusher.wsgi"]
