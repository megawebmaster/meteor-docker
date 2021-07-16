FROM ubuntu:bionic

# Install Meteor with specfic version and remove leftovers
ENV METEOR_VERSION 2.3.2
RUN apt-get update && \
		apt-get install -y curl python build-essential git && \
		export METEOR_ALLOW_SUPERUSER=true && \
		curl https://install.meteor.com/?release=${METEOR_VERSION} | sed s/--progress-bar/-sL/g | sh && \
		apt-get remove -y curl python build-essential git && \ 
		rm -rf /var/lib/apt/lists/*

# Expose Meteor Node and npm binaries
RUN export "NODE=$(find /root/.meteor/ -path '*bin/node' | grep '/root/.meteor/packages/meteor-tool/' | sort | head -n 1)" && \
		ln -sf ${NODE} /usr/local/bin/node && \
		ln -sf "$(dirname "$NODE")/npm" /usr/local/bin/npm

