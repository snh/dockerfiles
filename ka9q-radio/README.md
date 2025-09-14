# _ka9q-radio_ Containerised

A basic containerised version of [_ka9q-radio_](https://github.com/ka9q/ka9q-radio).

[_avahi-daemon_](https://avahi.org/) is required on the host running the container (included by default with Raspberry Pi OS).

## Example configurations

### `radiod.conf`

#### RTL-SDR

```ini
[global]
hardware = rtlsdr
mode = fm
status = sonde.local
iface = lo
ttl = 0
data = sonde-pcm.local

[rtlsdr]
device = rtlsdr
samprate = 2048000
frequency = 401m5
agc = 1
```

#### Airspy Mini

```ini
[global]
hardware = airspy
mode = fm
status = sonde.local
iface = lo
ttl = 0
data = sonde-pcm.local

[airspy]
device = airspy
frequency = 405m6
```

### Docker Compose

```yaml
services:
  ka9q-radio:
    build: https://github.com/snh/dockerfiles.git#main:ka9q-radio
    cap_add:
      - CAP_NET_ADMIN
      - CAP_SYS_NICE
    container_name: ka9q-radio
    devices:
      - "/dev/bus/usb:/dev/bus/usb"
    network_mode: host
    restart: always
    tty: true
    volumes:
      - /opt/ka9q-radio/radiod.conf:/etc/radio/radiod.conf:ro
      - /opt/ka9q-radio/data:/var/lib/ka9q-radio
      - /var/run/dbus:/var/run/dbus
```

## Example commands

### Build the container

```shell
docker build -t ka9q-radio https://github.com/snh/dockerfiles.git#main:ka9q-radio
```

### Run `radiod` (the default application in the container)

```shell
docker run --name ka9q-radio --rm -it --network=host --device=/dev/bus/usb \
--cap-add CAP_NET_ADMIN --cap-add CAP_SYS_NICE \
-v /opt/ka9q-radio/radiod.conf:/etc/radio/radiod.conf:ro \
-v /opt/ka9q-radio/data:/var/lib/ka9q-radio \
-v /var/run/dbus:/var/run/dbus \
ka9q-radio
```

### Generate FFT wisdom file

Run this command after running `radiod` at least once for a radiosonde decode cycle, which will populate `/var/lib/ka9q-radio/fft.log` with FFT transforms (one per line), which can then be used to determine which transforms need to be generated.

This command will take a while to run, depending on how many unique FFT transforms are in the log file.

```shell
docker run --name ka9q-radio-fft --rm -it --network=host \
-v /opt/ka9q-radio/data:/var/lib/ka9q-radio \
ka9q-radio /bin/sh -c "cat /var/lib/ka9q-radio/fft.log | sort | uniq | fft-gen -v -v"
```

You should re-run this command periodically to add any new FFT transforms to the wisdom file. Remembering that `radiod` will need to be restarted to pick up any changes to the wisdom file.

## Miscellaneous notes

### _radiosonde_auto_rx_

Follow the great ka9q-radio notes by [projecthorus/radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki/KA9Q%E2%80%90Radio-Setup-Notes), making sure to include `/var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket` in the volumes when launching the _radiosonde_auto_rx_ container.

```shell
docker run --rm -it -v /opt/auto_rx/station.cfg:/opt/auto_rx/station.cfg:ro --network=host -v /var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket ghcr.io/projecthorus/radiosonde_auto_rx:latest
```

```yaml
services:
  radiosonde_auto_rx:
    container_name: radiosonde_auto_rx
    image: ghcr.io/projecthorus/radiosonde_auto_rx:latest
    network_mode: host
    ports:
      - "5000:5000"
    restart: always
    volumes:
      - /opt/auto_rx/station.cfg:/opt/auto_rx/station.cfg:ro
      - /opt/auto_rx/logs:/opt/auto_rx/log
      - type: bind
        source: /var/run/avahi-daemon/socket
        target: /var/run/avahi-daemon/socket
```

The earlier `radiod.conf` examples use the following snippets of config in `station.cfg`, which should be adjusted to suit your needs in combination with adjustments to `radiod.conf`.

```ini
[sdr]
sdr_type = KA9Q
sdr_quantity = 4
sdr_hostname = sonde.local

[search_params]
min_freq = 400.5 # 400.05 for Airspy Mini
max_freq = 402.5 # 403.00 for Airspy Mini
```
