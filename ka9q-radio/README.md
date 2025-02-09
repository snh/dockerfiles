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
