# ZombieLoad PoC

This repository contains several applications, demonstrating ZombieLoad. 

<img src="https://github.com/zombieload/zombieload/raw/master/demo.gif" width="800"> 

## Proof of Concepts

This repository contains two different proof-of-concept attacks showing ZombieLoad. It also includes four different victim applications to test the leakage in various scenarios. 

All demos are tested with an Intel Core i7-8650U, but they should work on any Linux system with any modern Intel Core or Xeon CPU since 2011. 
We provide one variant for Linux, which we tested on Ubuntu 18.04.1 LTS, and one variant for Windows, which we tested on Windows 10 (1803 build 17134.706). 

For best results, we recommend a fast CPU. 

## Building

The PoCs only require GCC and Make (on Linux) or MinGW-w64 (on Windows) to compile. 

Building an attacker or a victim is as simple as running `make` in the folder of the application. 


## Attacker Variants

The repository contains two different attacker variants.

### Variant 1 (Linux only)

Variant 1 is the fastest, easiest and most stable variant for a privileged attacker (i.e., it requires root privileges). Hence, except for testing, this is especially useful for attacks on SGX or for attacks on virtual machines. 

##### Run

For this variant, KASLR and KPTI have to be disabled. This can be achieved by providing `nopti nokaslr` to the kernel command line. 
Then, run the attacker on one hyperthread as root: `sudo taskset -c 3 ./leak`

### Variant 2 (Windows only)

Variant 2 does not require privileges but it only works on Windows. 

##### Run
Run the attacker on one hyperthread: `taskset -c 3 ./leak`. It takes a while (up to 1 minute) until the leakage starts, as the PoC has to wait for Windows to collect information about the memory used by the PoC. Starting a different program which uses memory (e.g., a browser) sometimes reduces the waiting time. 


## Victim Applications

All attacker variants can be used to leak data from the following victim applications. All victim applications leak one uppercase letter. Independent of the chosen victim and attacker application, the attacker displays a histogram of leaked values. 

An example output is as follows (for the secret letter 'X' loaded by the victim).

    A: (   0) 
    B: (   0) 
    C: (   0) 
    D: (   0) 
    E: (   1) 
    F: (   0) 
    G: (   2) 
    H: (   0) 
    I: (   0) 
    J: (   0) 
    K: (   0) 
    L: (   0) 
    M: (   0) 
    N: (   0) 
    O: (   0) 
    P: (  12) 
    Q: (   1) 
    R: (   1) 
    S: (   0) 
    T: (   0) 
    U: (   2) 
    V: (   1) 
    W: (   0) 
    X: (1303) ############################################################
    Y: (   0) 
    Z: (   1) 

### Userspace Victim (Linux and Windows)

An unprivileged user application which constantly loads the same value from its memory. 

##### Run

Simply run the victim on the same physical core but a different hyperthread as the attacker: `taskset -c 7 ./secret`. You can also provide a secret letter to the victim application as a parameter, e.g., `taskset -c 7 ./secret B` to access memory containing 'B's. The default secret letter is 'X'. 

As soon as the victim is started, there should be a clear signal in the attacker process, i.e., the bar for the leaked letter should get longer. 

### Kernel Victim (Linux only)

A kernel module which constantly loads the letter 'J'.

##### Run

Before running the victim, the kernel module has to be loaded into the kernel. This is done by running `sudo insmod leaky.ko`. Then, simply run the victim on the same physical core but a different hyperthread as the attacker: `taskset -c 7 ./secret`.

As soon as the victim is started, there should be a clear signal in the attacker process, i.e., the bar for the letter 'J' should get longer. 

### Intel SGX Victim (Linux only)

An Intel SGX enclave which constantly loads the letter 'S'. This victim requires that the SGX driver and SDK are installed. 

##### Run

Simply run the victim on the same physical core but a different hyperthread as the attacker: `taskset -c 7 ./secret`.

As soon as the victim is started, there should be a clear signal in the attacker process, i.e., the bar for the letter 'S' should get longer. 

### VM Victim (Linux and Windows)

A virtual machine containing an application which constantly loads the same value from its memory. This victim requires that QEMU is installed, and VT-x is enabled. 

##### Run

Simply run the victim on the same physical core but a different hyperthread as the attacker: `taskset -c 7 ./secret.sh`. 
As soon as the virtual machine started, the victim is run using  `secret X`
Where `X` is the secret character. There should be a clear signal in the attacker process, i.e., the bar for the leaked letter should get longer. 


## Frequently Asked Questions

* **How do I know which core IDs are hyperthreads?**

    On Linux, you can run `lscpu -e`. This gives you a list of logical cores and their corresponding physical core. Cores mapping to the same physical core are hyperthreads. 

* **Can I run the PoC in a virtual machine?**

    Yes, the PoC also works on virtual machines. However, due to the additional layer introduced by a virtual machine, it might not work as good as on native hardware. 

* **It just does not work on my computer, what can I do?**

    There can be a lot of different reasons for that. We collected a few things you can try:
    
    * Ensure that your CPU frequency is at the maximum, and frequency scaling is disabled.
    * If you run it on a mobile device (e.g., a laptop), ensure that it is plugged in to get the best performance.
    * Try to pin the tools to a specific CPU core (e.g. with taskset). Also try different cores and core combinations. Leaking values only works if attacker and victim run on the same physical core. 
    * Vary the load on your computer. On some machines it works better if the load is higher, on others it works better if the load is lower.
    * Try to restart the demos and also your computer. Especially after a standby, the timings are broken on some computers. 

## Warnings
**Warning #1**: We are providing this code as-is. You are responsible for protecting yourself, your property and data, and others from any risks caused by this code. This code may cause unexpected and undesirable behavior to occur on your machine. This code may not detect the vulnerability on your machine.

**Warning #2**: If you find that a computer is susceptible to ZombieLoad, you may want to avoid using it as a multi-user system. ZombieLoad breaches the CPU's memory protection. On a machine that is susceptible to ZombieLoad, one process can potentially read all data used by other processes or by the kernel.

**Warning #3**: This code is only for testing purposes. Do not run it on any productive systems. Do not run it on any system that might be used by another person or entity.
