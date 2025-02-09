# -------------------
# The build container
# -------------------
FROM debian:bookworm-slim AS build

# ka9q-radio git ref to use
ARG REF=main

# Install necessary packages for building the application
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    avahi-utils \
    build-essential \
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

# Download the source code into the build container
ADD https://github.com/ka9q/ka9q-radio/archive/refs/heads/$REF.zip /tmp/ka9q-radio.zip
RUN unzip /tmp/ka9q-radio.zip -d /tmp && \
  mv /tmp/ka9q-radio-* /src && \
  rm /tmp/ka9q-radio.zip

# Set the working directory
WORKDIR /src

# Build the application using the provided Makefile
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
    libogg0 \
    libopus0 \
    libportaudio2 \
    libsamplerate0 \
    rtl-sdr \
    tini && \
    rm -rf /var/lib/apt/lists/*

# Copy the built binaries from the build stage
COPY --from=build \
    /src/aprs \
    /src/aprsfeed \
    /src/cwd \
    /src/opusd \
    /src/packetd \
    /src/radiod \
    /src/rdsd \
    /src/stereod \
    /usr/local/sbin/

# Copy additional binaries from the build stage
COPY --from=build \
    /src/control \
    /src/jt-decoded \
    /src/metadump \
    /src/monitor \
    /src/opussend \
    /src/pcmcat \
    /src/pcmrecord \
    /src/pcmsend \
    /src/pcmspawn \
    /src/pl \
    /src/powers \
    /src/setfilt \
    /src/show-pkt \
    /src/show-sig \
    /src/tune \
    /src/wd-record \
    /usr/local/bin/

# Copy shared resources from the build stage
COPY --from=build /src/share /usr/local/share/ka9q-radio

# Create necessary directories
RUN mkdir -p /etc/radio /etc/fftw /var/lib/ka9q-radio

# Use tini as the init system to handle reaping zombie processes and signal forwarding
ENTRYPOINT ["/usr/bin/tini", "--"]

# Define the default command to run the application
CMD ["/usr/local/sbin/radiod", "/etc/radio/radiod.conf"]
