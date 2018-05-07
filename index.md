# Hands-on LAB - Pervasive Encryption on LinuxONE
As of March 2018, the LinuxONE has two categories of crypto hardware.
- CPACF – Provides support for symmetric ciphers and secure hash algorithms (SHA) on every central processor. The potential encryption/decryption throughput scales with the number of CPs.
- CEX6S – The Crypto Express feature traditionally could be configured in two ways: Either as cryptographic Coprocessor (CEXC) for secure key encrypted transactions, or as cryptographic Accelerator (CEXA) for Secure Sockets Layer (SSL) acceleration. A CEXA works in clear key mode. The Crypto Express 6S allows for a third mode as a Secure IBM CCA Coprocessor.

## LinuxONE Crypto Background
OpenSSL needs the engine ibmca to communicate with the interface library (libICA). The libICA library then communicates with CPACF or via the Linux generic device driver z90crypt with a CEX (if available). The device driver z90crypt must be loaded in order to use CEX features.

We know many potential exploiters, and not limited to the following list:
- WebSphere Application Server/Portal
- Java Applications
- IBM HTTP Server
- Apache
- WebSphere Plugin
- Linux SSH, SFTP , SCP
- In Kernel Crypto Exploiters
- DM-Crypt
- IPSec
...

## LinuxONE Crypto Stack
<crypto stack picture here>
  
## Hands-on LAB Agenda
### Part I - Enabling Linux to use the Hardware
#### CPACF Enablement verification
A Linux on IBM Z user can easily check whether the Crypto Enablement feature is installed
and which algorithms are supported in hardware. Hardware-acceleration for DES, TDES,
AES, and GHASH requires CPACF.
Issue the following command /proc/cpuinfo to discover whether the CPACF feature is enabled
on your hardware.
```
root@crypt06:~# cat /proc/cpuinfo 
vendor_id       : IBM/S390
# processors    : 2
bogomips per cpu: 21881.00
features	: esan3 zarch stfle msa ldisp eimm dfp edat etf3eh highgprs te vx 
cache0          : level=1 type=Data scope=Private size=128K line_size=256 associativity=8
cache1          : level=1 type=Instruction scope=Private size=128K line_size=256 associativity=8
cache2          : level=2 type=Data scope=Private size=4096K line_size=256 associativity=8
cache3          : level=2 type=Instruction scope=Private size=2048K line_size=256 associativity=8
cache4          : level=3 type=Unified scope=Shared size=131072K line_size=256 associativity=32
cache5          : level=4 type=Unified scope=Shared size=688128K line_size=256 associativity=42
processor 0: version = FF,  identification = 233EF7,  machine = 3906
processor 1: version = FF,  identification = 233EF7,  machine = 3906
```
#### Installing libica 3.0
To make use of the libica hardware support for cryptographic functions, you must install the libica version 3.0 package. Obtain the current libica version from your distribution provider for automated installation. Please issue the following command:
```
root@crypt06:~# sudo apt-get install libica-utils
```

After the libica utility is installed, use the icaiinfo command to check on the CPACF feature code enablement. If the Crypto Enablement feature 3863 is installed, you will see that besides SHA, other algorithms are available with hardware support. The icainfo command displays which CPACF functions are supported by the implementation inside the libica library. Issue the following command to show that the device driver loaded how which cryptographic algorithms will be accelerated and hardware or software way.
```
[root@ghrhel74crypt ~]# icainfo
      Cryptographic algorithm support      
-------------------------------------------
 function      |  hardware  |  software  
---------------+------------+------------
         SHA-1 |    yes     |     yes
       SHA-224 |    yes     |     yes
       SHA-256 |    yes     |     yes
       SHA-384 |    yes     |     yes
       SHA-512 |    yes     |     yes
      SHA3-224 |    yes     |      no
      SHA3-256 |    yes     |      no
      SHA3-384 |    yes     |      no
      SHA3-512 |    yes     |      no
     SHAKE-128 |    yes     |      no
     SHAKE-256 |    yes     |      no
         GHASH |    yes     |      no
         P_RNG |    yes     |     yes
  DRBG-SHA-512 |    yes     |     yes
        RSA ME |    yes     |     yes
       RSA CRT |    yes     |     yes
       DES ECB |    yes     |     yes
       DES CBC |    yes     |     yes
       DES OFB |    yes     |      no
       DES CFB |    yes     |      no
       DES CTR |    yes     |      no
      DES CMAC |    yes     |      no
      3DES ECB |    yes     |     yes
      3DES CBC |    yes     |     yes
      3DES OFB |    yes     |      no
      3DES CFB |    yes     |      no
      3DES CTR |    yes     |      no
     3DES CMAC |    yes     |      no
       AES ECB |    yes     |     yes
       AES CBC |    yes     |     yes
       AES OFB |    yes     |      no
       AES CFB |    yes     |      no
       AES CTR |    yes     |      no
      AES CMAC |    yes     |      no
       AES XTS |    yes     |      no
       AES GCM |    yes     |      no
-------------------------------------------
No built-in FIPS support.
```

From the cpuinfo output, you can find the features that are enabled in the central processors.
If the features list has msa listed, it means that CPACF is enabled.

Most of the distributions include a generic kernel image for the specific platform. These
device drivers for the generic kernel image are included as loadable kernel modules because
statically compiling many drivers into one kernel causes the kernel image to be much larger.
This kernel might be too large to boot on computers with limited memory.

#### Starting crypto module
Use the lsmod command to check whether the crypto device driver module is already loaded.
If the module is not loaded, use the modprobe command to load the device driver module.
If it shows that the Linux system is not yet loaded with the crypto device driver
modules, so you must load it manually. The cryptographic device driver consists of multiple,
separate modules.
```
root@crypt06:~# modprobe aes_s390
root@crypt06:~# modprobe des_s390
root@crypt06:~# modprobe sha1_s390
root@crypt06:~# modprobe sha256_s390
root@crypt06:~# modprobe sha512_s390
root@crypt06:~# modprobe rng     
root@crypt06:~# modprobe prng
root@crypt06:~# modprobe hmac
```

#### Pervasive Encryption readiness assessment
Validate that all the crypto module are properly loaded. Please issue the following command:
```
root@crypt06:~# lsmod | grep s390
ghash_s390             16384  0
aes_s390               20480  0
des_s390               16384  0
des_generic            28672  1 des_s390
sha512_s390            16384  0
sha256_s390            16384  0
sha1_s390              16384  0
sha_common             16384  3 sha256_s390,sha1_s390,sha512_s390
```

Validate that the libica crypto API is working properly. Please issue the following command:
```
root@crypt06:~# icastats
 function     |          # hardware      |       # software
--------------+--------------------------+-------------------------
              |       ENC    CRYPT   DEC |        ENC    CRYPT   DEC
--------------+--------------------------+-------------------------
        SHA-1 |               0          |                0
      SHA-224 |               0          |                0
      SHA-256 |               0          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |             169          |                0
       RSA-ME |               0          |                0
      RSA-CRT |               0          |                0
      DES ECB |         0              0 |         0             0
      DES CBC |         0              0 |         0             0
      DES OFB |         0              0 |         0             0
      DES CFB |         0              0 |         0             0
      DES CTR |         0              0 |         0             0
     DES CMAC |         0              0 |         0             0
     3DES ECB |         0              0 |         0             0
     3DES CBC |         0              0 |         0             0
     3DES OFB |         0              0 |         0             0
     3DES CFB |         0              0 |         0             0
     3DES CTR |         0              0 |         0             0
    3DES CMAC |         0              0 |         0             0
      AES ECB |         0              0 |         0             0
      AES CBC |         0              0 |         0             0
      AES OFB |         0              0 |         0             0
      AES CFB |         0              0 |         0             0
      AES CTR |         0              0 |         0             0
     AES CMAC |         0              0 |         0             0
      AES XTS |         0              0 |         0             0
```

Your hands-on LAB environment is now properly setup!

### Pervasive Encryption - Enabling OpenSSL and openSSH to use the Hardware

### Pervasive Encryption - Enabling dm-crypt to use the Hardware

Thank you, and see you soon.
