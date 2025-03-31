# -------------------
# The build container
# -------------------
FROM debian:bookworm-slim AS build

# git refs to use
ARG RTL_SDR_REF=refs/tags/v2.0.2
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
    libncurses5-dev \
    libncursesw5-dev \
    libogg-dev \
    libopus-dev \
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

# Download the rtl-sdr source code into the build container
ADD https://github.com/osmocom/rtl-sdr/archive/$RTL_SDR_REF.zip /tmp/rtl-sdr.zip
RUN unzip /tmp/rtl-sdr.zip -d /tmp && \
    mv /tmp/rtl-sdr-* /src/rtl-sdr && \
    rm /tmp/rtl-sdr.zip && \
    mkdir -p /src/rtl-sdr/build

# Set the working directory for the rtl-sdr build
WORKDIR /src/rtl-sdr/build

# Build the rtl-sdr source code, and install to the build container
# as well as a staging directory for the install to the later 
# application container
RUN cmake -Wno-dev .. && \
    make && \
    make install && \
    make DESTDIR=/src/rtl-sdr/install install
    
# Download the ka9q-radio source code into the build container
ADD https://github.com/ka9q/ka9q-radio/archive/$KA9Q_RADIO_REF.zip /tmp/ka9q-radio.zip
RUN unzip /tmp/ka9q-radio.zip -d /tmp && \
  mv /tmp/ka9q-radio-* /src/ka9q-radio && \
  rm /tmp/ka9q-radio.zip

# Set the working directory for the ka9q-radio build
WORKDIR /src/ka9q-radio

# Build ka9q-radio using the provided Linux Makefile
RUN make -f Makefile.linux

# -------------------------
# The application container
# -------------------------
FROM debian:bookworm-slim

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
    libiniparser1 \
    libncurses5 \
    libncursesw5 \
    libogg0 \
    libopus0 \
    libportaudio2 \
    libsamplerate0 \
    tini && \
    rm -rf /var/lib/apt/lists/*

# Copy over the rtl-sdr install from the build stage
COPY --from=build /src/rtl-sdr/install /
RUN ldconfig

# Copy the built binaries from the build stage
COPY --from=build \
    /src/ka9q-radio/aprs \
    /src/ka9q-radio/aprsfeed \
    /src/ka9q-radio/cwd \
    /src/ka9q-radio/packetd \
    /src/ka9q-radio/radiod \
    /usr/local/sbin/

# Copy additional binaries from the build stage
COPY --from=build \
    /src/ka9q-radio/control \
    /src/ka9q-radio/jt-decoded \
    /src/ka9q-radio/metadump \
    /src/ka9q-radio/monitor \
    /src/ka9q-radio/pcmrecord \
    /src/ka9q-radio/pl \
    /src/ka9q-radio/powers \
    /src/ka9q-radio/tune \
    /src/ka9q-radio/wd-record \
    /usr/local/bin/

# Copy shared libraries from the build stage
COPY --from=build /src/ka9q-radio/*.so /usr/local/lib/ka9q-radio/

# Copy shared resources from the build stage
COPY --from=build /src/ka9q-radio/share /usr/local/share/ka9q-radio

# Create necessary directories
RUN mkdir -p /etc/radio /etc/fftw /var/lib/ka9q-radio

# Use tini as the init system to handle reaping zombie processes and signal forwarding
ENTRYPOINT ["/usr/bin/tini", "--"]

# Define the default command to run the application
CMD ["/usr/local/sbin/radiod", "/etc/radio/radiod.conf"]
