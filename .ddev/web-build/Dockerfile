## #ddev-generated
## You can copy this Dockerfile.example to Dockerfile to add configuration
## or packages or anything else to your webimage
## These additions will be appended last to DDEV's own Dockerfile
RUN echo "deb [signed-by=/etc/apt/keyrings/hubzero-signing-key.asc] http://packages.hubzero.org/deb julian-deb8 main" > /etc/apt/sources.list.d/hubzero.list

RUN curl -L https://packages.hubzero.org/deb/hubzero-signing-key.asc -o /etc/apt/keyrings/hubzero-signing-key.asc

RUN apt-get update
RUN apt-get install -y hubzero-solr
