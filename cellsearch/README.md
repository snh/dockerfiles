# _cellsearch_ Containerised

A basic containerised version of _cellsearch_ from the [_LTE-Cell-Scanner_](https://github.com/Evrytania/LTE-Cell-Scanner) project.

Useful for determining a suitable PPM value for a SDR. Thank you @darksidelemm for the [great notes](https://gist.github.com/darksidelemm/b517e6a9b821c50c170f1b9b7d65b824) that got me started.

## Example commands

### Build the container

```shell
docker build -t cellsearch --no-cache https://github.com/snh/dockerfiles.git#main:cellsearch
```

### Run a search over 778Mhz

```shell
docker run --rm -it --device=/dev/bus/usb cellsearch --freq-start 778e6 --freq-end 778e6
```

## Converting CrystalCorrectionFactor to PPM

In `python`, run the following, substituting `CrystalCorrectionFactor`:

```shell
1e6*(1-CrystalCorrectionFactor)
```
