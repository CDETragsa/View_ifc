FROM ubuntu:latest

WORKDIR /
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get -y update && apt-get -y --no-install-recommends --no-install-suggests install python3 python3-pip unzip wget libpq-dev build-essential libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev python3-setuptools python3-dev python3-wheel supervisor libjpeg-dev
RUN wget -qO- https://nodejs.org/dist/v16.18.0/node-v16.18.0-linux-x64.tar.gz | tar -xzC /opt
ENV PATH /opt/node-v16.18.0-linux-x64/bin:$PATH
RUN npm install -g jsdoc gltf-pipeline

RUN wget -O /tmp/ifcopenshell_python.zip https://s3.amazonaws.com/ifcopenshell-builds/ifcopenshell-python-`python3 -c 'import sys;print("".join(map(str, sys.version_info[0:2]))[0:2])'`-v0.7.0-b5133c6-linux64.zip
RUN mkdir -p `python3 -c 'import site; print(site.getusersitepackages())'`
RUN unzip -d `python3 -c 'import site; print(site.getusersitepackages())'` /tmp/ifcopenshell_python.zip

# Server
WORKDIR /www
COPY application/*.py application/*.txt application/config.json application/config.schema /www/
COPY application/templates /www/templates

# Python dependencies
RUN python3 -m pip install -r requirements-production.txt

# comenta las siguientes líneas para uso de render.com
#COPY .git/HEAD /tmp/.git/HEAD
#COPY .git/refs/ /tmp/.git/refs/
#RUN /bin/bash -c '(cat /tmp/.git/$(cat /tmp/.git/HEAD | cut -d \  -f 2)) || cat /tmp/.git/HEAD' > /version
#RUN sed -i "4i<script>console.log('pipeline version: $(cat /version)');</script>" /www/templates/*.html

#Reemplázalas con una instrucción que cree un archivo /version vacío o con un valor predeterminado
RUN echo "unknown" > /version

# Reemplaza las líneas problemáticas con una instrucción que use una variable
# Agrega esta línea:
#ARG COMMIT_HASH=unknown
ARG COMMIT_HASH=4d26c33f79a2b67bd86d9f3f48d1ee7e8333c625
RUN echo "$COMMIT_HASH" > /version
RUN sed -i "4i<script>console.log('pipeline version: $(cat /version)');</script>" /www/templates/*.html

RUN rm -rf /tmp/.git

COPY application/static /www/static/
COPY application/bimsurfer /www/bimsurfer

WORKDIR /www/static
RUN npm i && npx webpack --env postfix=$(cat /version) && rm -rf node_modules

COPY application/queue.conf /etc/supervisord.conf
RUN sed -i s/NUM_WORKERS/`python3 -c "import json; print(json.load(open('/www/config.json'))['performance']['num_workers'])"`/g /etc/supervisord.conf

WORKDIR /www
