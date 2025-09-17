# -------------------
# The build container
# -------------------
FROM ka9q-radio:build AS build

# git refs to use
ARG ONION_REF=refs/heads/master
ARG KA9Q_WEB_REF=refs/heads/main

# Install necessary packages for building the application
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libgnutls28-dev \
    libgcrypt-dev \
    && rm -rf /var/lib/apt/lists/*

# Download the onion source code into the build container
ADD https://github.com/davidmoreno/onion/archive/$ONION_REF.zip /tmp/onion.zip
RUN unzip /tmp/onion.zip -d /tmp && \
    mv /tmp/onion-* /src/onion && \
    rm /tmp/onion.zip && \
    mkdir -p /src/onion/build

# Set the working directory for the onion build
WORKDIR /src/onion/build

# Build the onion source code, and install to the build container
RUN cmake -DONION_USE_PAM=false -DONION_USE_PNG=false \
      -DONION_USE_JPEG=false -DONION_USE_XML2=false \
      -DONION_USE_SYSTEMD=false -DONION_USE_SQLITE3=false \
      -DONION_USE_REDIS=false -DONION_USE_GC=false -DONION_USE_TESTS=false \
      -DONION_EXAMPLES=false -DONION_USE_BINDINGS_CPP=false .. && \
    make && \
    make install

# Download the ka9q-web source code into the build container
ADD https://github.com/wa2n-code/ka9q-web/archive/$KA9Q_WEB_REF.zip /tmp/ka9q-web.zip
RUN unzip /tmp/ka9q-web.zip -d /tmp && \
    mv /tmp/ka9q-web-* /src/ka9q-web && \
    rm /tmp/ka9q-web.zip

# Set the working directory for the ka9q-web build
WORKDIR /src/ka9q-web

# Build ka9q-web using the provided Linux Makefile
RUN make

# -------------------------
# The application container
# -------------------------
FROM debian:trixie-slim

# Install necessary packages for running the application
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libgcrypt20 \
    libgnutls30t64 \
    tini && \
    rm -rf /var/lib/apt/lists/*

# Copy the built binary from the build stage
COPY --from=build /src/ka9q-web/ka9q-web /usr/local/sbin/

# Copy shared libraries from the build stage
COPY --from=build /usr/local/lib/libonion.so /usr/local/lib/

# Copy shared resources from the build stage
COPY --from=build /src/ka9q-web/html /usr/local/share/ka9q-web

# Update the shared library cache
RUN ldconfig

# Use tini as the init system to handle reaping zombie processes and signal forwarding
ENTRYPOINT ["/usr/bin/tini", "--"]

# Define the default command to run the application
CMD ["/usr/local/sbin/ka9q-web"]
