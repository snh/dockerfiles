# -------------------
# The build container
# -------------------
FROM debian:trixie-slim AS build

# git refs to use
ARG RFONE_HOST_REF=v1.0.2
ARG KA9Q_RADIO_REF=refs/heads/main

# Install necessary packages for building the application
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    avahi-utils \
    build-essential \
    cmake \
    gcc \
    git \
    libairspy-dev \
    libairspyhf-dev \
    libasound2-dev \
    libavahi-client-dev \
    libbsd-dev \
    libfftw3-dev \
    libhackrf-dev \
    libiniparser-dev \
    libncurses-dev \
    libogg-dev \
    libopus-dev \
    librtlsdr-dev \
    libsamplerate0-dev \
    libusb-1.0-0-dev \
    libusb-dev \
    make \
    portaudio19-dev \
    rsync \
    time \
    unzip \
    uuid-dev \
    && rm -rf /var/lib/apt/lists/*

# Create a directory for source code
RUN mkdir -p /src

# Download the rfone_host source code into the build container
ADD https://github.com/hydrasdr/rfone_host/archive/$RFONE_HOST_REF.zip /tmp/rfone_host.zip
RUN unzip /tmp/rfone_host.zip -d /tmp && \
    mv /tmp/rfone_host-* /src/rfone_host && \
    rm /tmp/rfone_host.zip && \
    mkdir -p /src/rfone_host/build

# Set the working directory for the rfone_host build
WORKDIR /src/rfone_host/build

# Build the rfone_host source code, and install to the build container
RUN cmake .. && \
    make && \
    make install

# Download the ka9q-radio source code into the build container
ADD https://github.com/ka9q/ka9q-radio/archive/$KA9Q_RADIO_REF.zip /tmp/ka9q-radio.zip
RUN unzip /tmp/ka9q-radio.zip -d /tmp && \
    mv /tmp/ka9q-radio-* /src/ka9q-radio && \
    rm /tmp/ka9q-radio.zip

# Set the working directory for the ka9q-radio build
WORKDIR /src/ka9q-radio

# Build ka9q-radio using the provided Linux Makefile
RUN make

# -------------------------
# The application container
# -------------------------
FROM debian:trixie-slim

# Install necessary packages for running the application
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    avahi-utils \
    libairspy0 \
    libairspyhf1 \
    libasound2 \
    libbsd0 \
    libfftw3-bin \
    libfftw3-single3 \
    libhackrf0 \
    libiniparser4 \
    libncursesw6 \
    libogg0 \
    libopus0 \
    libportaudio2 \
    librtlsdr0 \
    libsamplerate0 \
    tini && \
    rm -rf /var/lib/apt/lists/*

# Copy the built binaries from the build stage
COPY --from=build \
  /src/ka9q-radio/src/aprs \
  /src/ka9q-radio/src/aprsfeed \
  /src/ka9q-radio/src/cwd \
  /src/ka9q-radio/src/packetd \
  /src/ka9q-radio/src/radiod \
  /usr/local/sbin/

# Copy additional binaries from the build stage
COPY --from=build \
  /src/ka9q-radio/src/control \
  /src/ka9q-radio/src/fft-gen \
  /src/ka9q-radio/src/jt-decoded \
  /src/ka9q-radio/src/metadump \
  /src/ka9q-radio/src/monitor \
  /src/ka9q-radio/src/pcmrecord \
  /src/ka9q-radio/src/pl \
  /src/ka9q-radio/src/powers \
  /src/ka9q-radio/src/tune \
  /src/ka9q-radio/src/wd-record \
  /usr/local/bin/

# Copy shared libraries from the build stage
COPY --from=build /src/ka9q-radio/src/*.so /usr/local/lib/ka9q-radio/

# Copy shared resources from the build stage
COPY --from=build /src/ka9q-radio/share /usr/local/share/ka9q-radio

# Create necessary directories
RUN mkdir -p /etc/radio /etc/fftw /var/lib/ka9q-radio

# Use tini as the init system to handle reaping zombie processes and signal forwarding
ENTRYPOINT ["/usr/bin/tini", "--"]

# Define the default command to run the application
CMD ["/usr/local/sbin/radiod", "/etc/radio/radiod.conf"]
