# -------------------
# The build container
# -------------------
  FROM debian:bookworm-slim AS build

  RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    libblas-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libfftw3-dev \
    libitpp-dev \
    liblapack-dev \
    libncurses5-dev \
    librtlsdr-dev \
    unzip && \
    rm -rf /var/lib/apt/lists/*

  COPY cellsearch.patch /root/cellsearch.patch
  ADD https://github.com/Evrytania/LTE-Cell-Scanner/archive/refs/heads/master.zip /root/LTE-Cell-Scanner-master.zip
  RUN unzip /root/LTE-Cell-Scanner-master.zip -d /root && \
    rm /root/LTE-Cell-Scanner-master.zip && \
    cd /root/LTE-Cell-Scanner-master && \
    patch -p1 -i /root/cellsearch.patch && \
    mkdir build && \
    cd build && \
    cmake ../ && \
    make CellSearch && \
    rm -rf /root/rtl-ais-master

  # -------------------------
  # The application container
  # -------------------------
  FROM debian:bookworm-slim

  RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libboost-chrono1.74.0 \
    libboost-date-time1.74.0 \
    libboost-thread1.74.0 \
    libfftw3-double3 \
    libitpp8v5 \
    liblapack3 \
    libncurses6 \
    librtlsdr0 && \
    rm -rf /var/lib/apt/lists/*

  COPY --from=build /root/LTE-Cell-Scanner-master/build/src/CellSearch /usr/local/bin/CellSearch

  ENTRYPOINT ["/usr/local/bin/CellSearch"]
