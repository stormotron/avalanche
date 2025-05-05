Avalanche version 0.0.1
============================
Copyright (c) Netstorm, 2025

Avalanche is freeware for Linux designed to fill the kernel entropy pool with random numbers obtained from the network card or from the sound card (noise).

**Docker:**

When starting a Docker container, you must specify the source of the random numbers:
```
- RANDOM_SOURCE - network or sound;
- DEVICE_NAME - the name of the device to receive the random numbers;
- NETWORK_FILTER - filter for network mode.
```

In case of receiving data from network, packets will be captured on the DEVICE_NAME interface (e.g. eth1) if specified - otherwise the first available one will be used. Then an asymmetric Hash will be obtained from the captured packets and sent to the kernel as a source of random numbers. If you are using sound, take care of noise from your audio card whose name is selected with DEVICE_NAME (e.g. hw:0,0), the raw data will be sent directly to the kernel pool.

**Usage Examples:**

Network mode:
```
# docker run --net=host --privileged -e RANDOM_SOURCE=network -e NETWORK_FILTER=udp --name avalanche -d stormotron/avalanche:latest
```

Sound card mode:
```
# docker run --net=host --privileged -e RANDOM_SOURCE=sound --name avalanche -d stormotron/avalanche:latest
```

**Benchmarking:**
```
# dd if=/dev/random of=/dev/null status=progress
```

The program is only shipped as a binary image for Docker, on an As-Is basis.
