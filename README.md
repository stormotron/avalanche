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

Perhaps you are wondering how Avalanche is better than, for example, RdRand? The important thing to understand is not actually the implementation of the instruction itself, but what is the source of the Noise in your case. If your noise source is thermal or similar, it is quite possible that you will encounter “monotonicity”, that is, when you can predict the next pseudo-random number on some time interval. Avalanche suggests that you use two sources of noise - your network traffic or data from a sound card to which you can connect, for example, an FM/AM receiver. So how do you evaluate the randomness of the numbers?

First, estimate the speed of your random number source:
```
# dd if=/dev/random of=/dev/null status=progress
```

Then save the result to a file of the desired length to analyze it for monotonicity:
```
# dd if=/dev/random of=test_file.rnd bs=<your_bs> count=<required_size> status=progress iflag=fullblock
```

Analyze with the dieharder utility, paying attention to the test result and p-values:
```
; required 60MB
# dieharder -d 0 -g 201 -f test_file.rnd
; required 500MB
# dieharder -d 1 -g 201 -f test_file.rnd
; required 600MB
# dieharder -d 2 -g 201 -f test_file.rnd
```

Here are two examples of obtaining random numbers

Intel Atom with Haveged:
```
# dieharder -d 0 -g 201 -f test_file_haveged_atom.rnd
#=============================================================================#
#            dieharder version 3.31.1 Copyright 2003 Robert G. Brown          #
#=============================================================================#
   rng_name    |           filename             |rands/second|
 file_input_raw|                   test_file.rnd|  1.03e+07  |
#=============================================================================#
        test_name   |ntup| tsamples |psamples|  p-value |Assessment
#=============================================================================#
   diehard_birthdays|   0|       100|     100|0.14793305|  PASSED  

```

Intel Atom with Avalanche/Soundcard:
```
dieharder -d 0 -g 201 -f test_file_snd.rnd 
#=============================================================================#
#            dieharder version 3.31.1 Copyright 2003 Robert G. Brown          #
#=============================================================================#
   rng_name    |           filename             |rands/second|
 file_input_raw|                   test_file.rnd|  1.06e+07  |
#=============================================================================#
        test_name   |ntup| tsamples |psamples|  p-value |Assessment
#=============================================================================#
   diehard_birthdays|   0|       100|     100|0.99376437|  PASSED  
```

As you can see, relative to p-values - the quality of the random data has increased significantly.

**The program is only shipped as a binary image for Docker, on an As-Is basis.**
