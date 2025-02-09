# _ka9q-radio_ Containerised

A basic containerised version of [_ka9q-radio_](https://github.com/ka9q/ka9q-radio).

[_avahi-daemon_](https://avahi.org/) is required on the host running the container (included by default with Raspberry Pi OS).

## Example configurations

### `radiod.conf` for a RTL-SDR

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
serial = 00000031
samprate = 2048000
frequency = 401m55
agc = 1
calibrate = -1e-6
```

### Docker Compose

```yaml
services:
  ka9q-radio:
    build:
      context: dockerfiles/ka9q-radio
    container_name: ka9q-radio
    devices:
      - "/dev/bus/usb:/dev/bus/usb"
    network_mode: host
    restart: always
    tty: true
    volumes:
      - /opt/ka9q-radio/radiod.conf:/etc/radio/radiod.conf:ro
      - /opt/ka9q-radio/wisdomf:/etc/fftw/wisdomf
      - /opt/ka9q-radio/wisdom:/var/lib/ka9q-radio/wisdom
      - /var/run/dbus:/var/run/dbus
```

## Example commands

### Build the container

```shell
docker build -t ka9q-radio <dockerfile-path>
```

### Run `fftwf-wisdom`

```shell
docker run --name ka9q-radio --rm -it --network=host --device=/dev/bus/usb \
-v /opt/ka9q-radio/radiod.conf:/etc/radio/radiod.conf:ro \
-v /opt/ka9q-radio/wisdomf:/etc/fftw/wisdomf \
-v /opt/ka9q-radio/wisdom:/var/lib/ka9q-radio/wisdom \
-v /var/run/dbus:/var/run/dbus \
ka9q-radio fftwf-wisdom -v -T 1 -w /var/lib/ka9q-radio/wisdom -o /etc/fftw/wisdomf cof51200
```

### Run `radiod` (the default application in the container)

```shell
docker run --name ka9q-radio --rm -it --network=host --device=/dev/bus/usb \
-v /opt/ka9q-radio/radiod.conf:/etc/radio/radiod.conf:ro \
-v /opt/ka9q-radio/wisdomf:/etc/fftw/wisdomf \
-v /opt/ka9q-radio/wisdom:/var/lib/ka9q-radio/wisdom \
-v /var/run/dbus:/var/run/dbus \
ka9q-radio
```

## Miscellaneous notes

### _radiosonde_auto_rx_

Follow the great ka9q-radio notes by [projecthorus/radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki/KA9Q%E2%80%90Radio-Setup-Notes), making sure to include `/var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket` in the volumes when launching the _radiosonde_auto_rx_ container.

```shell
docker run --rm -it -v /opt/auto_rx/station.cfg:/opt/auto_rx/station.cfg:ro --network=host -v /var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket ghcr.io/projecthorus/radiosonde_auto_rx:latest
```
