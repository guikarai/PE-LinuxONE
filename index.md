# Pervasive Encryption Applied - Hands-on LABs

Seen and used in IBM Systems Technical University
* l101813 Thu 10:15-11:15 D6 Barsac
* l101813 Thu 11:30-12:30 D6 Barsac  

# About Pervasive Encryption on LinuxONE
Pervasive encryption is a data-centric approach to information security that entails protecting data entering and exiting the z14 platform. It involves encrypting data in-flight and at-rest to meet complex compliance mandates and reduce the risks and financial losses of a data breach. It is a paradigm shift from selective encryption (where only the data that is required to achieve compliance is encrypted) to pervasive encryption. Pervasive encryption with z14 is enabled through tight platform integration that includes Linux on IBM Z or LinuxONE following features:
* Integrated cryptographic hardware: Central Processor Assist for Cryptographic Function (CPACF) is a co-processor on every processor unit that accelerates encryption. Crypto Express features can be used as hardware security modules (HSMs).
* Data set and file encryption: You can protect Linux file systems that is transparent to applications and databases.
* Network encryption: You can protect network data traffic by using standards-based encryption from endpoint to endpoint.

## LinuxONE Crypto Stack
Pervasive Encryption benefits of the full Power of Linux Ecosystem plus z14 Capabilities
* LUKS dm-crypt – Transparent file & volume encryption using industry unique CPACF protected-keys
* Network Security – Enterprise scale encryption and handshakes using z14 CPACF and SIMD (openSSL, IPSec...)

The IBM Z and LinuxONE systems provide cryptographic functions that, from an application program perspective, can be grouped as follows:
* Synchronous cryptographic functions, provided by the CP Assist for Cryptographic Function (CPACF) or the Crypto Express features when defined as an accelerator.
* Asynchronous cryptographic functions, provided by the Crypto Express features.

The IBM Z and LinuxONE systems provide also rich cryptographic functions available via a complete crypto stack made of a set of key crypto APIs.
![Image of the Crypto Stack](https://github.com/guikarai/PE-LinuxONE/blob/master/images/crypto-stack.png)

**Note:** Locate openSSL and dm-crypt. For the following, we will work on how set-up a Linux environment in order to benefit of Pervasive Encryption benefits.
  
## Enabling Linux to use the Hardware
### 1. CPACF Enablement verification
A Linux on IBM Z user can easily check whether the Crypto Enablement feature is installed and which algorithms are supported in hardware. Hardware-acceleration for DES, TDES, AES, and GHASH requires CPACF. Issue the command shown below to discover whether the CPACF feature is enabled
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

**Note**: msa on line 4, indicates that the CPACF instruction is properly supported and detected.

**Note 2**: vx on line 4, indicates that SIMD and vector instructions are properly supported and detected.

### 2. Installing libica
To make use of the libica hardware support for cryptographic functions, you must install the libica package. Obtain the current libica version from your distribution provider for automated installation by issuing the following command:
```
root@crypt06:~# sudo apt-get install libica-utils
```
After the libica utility is installed, use the **icainfo** command to check on the CPACF feature code enablement. 
The icainfo command displays which CPACF functions are supported by the implementation inside the libica library. Issue the following command to show which cryptographic algorithms will be hardware-accelerated by the libica driver, and which one will remain software-only implementations.

```
root@crypt06:~# icainfo
The following CP Assist for Cryptographic Function (CPACF) 
operations are supported by libica on this system:
 function      | # hardware | #software
---------------+------------+--------------
         SHA-1 |    yes     |     yes
       SHA-224 |    yes     |     yes
       SHA-256 |    yes     |     yes
       SHA-384 |    yes     |     yes
       SHA-512 |    yes     |     yes
         GHASH |    yes     |      no
         P_RNG |    yes     |     yes
  DRBG-SHA-512 |    yes     |     yes
        RSA ME |     no     |     yes
       RSA CRT |     no     |     yes
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
```

From the **cpuinfo** output, you can find the features that are enabled in the central processors. 
If the features list has msa listed, it means that CPACF is enabled. Most of the distributions include a generic kernel image for the specific platform. 
These device drivers for the generic kernel image are included as loadable kernel modules because statically compiling many drivers into one kernel causes the kernel image to be much larger. This kernel might be too large to boot on computers with limited memory.

### 3. Loading crypto modules
Let's use the **modprobe** command to load the device driver modules. Initially the Linux system is not yet configured to use  the crypto device driver modules, so you must load them manually. The cryptographic device drivers consists of multiple,
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

### 4. Pervasive Encryption readiness assessment
Validate that all the crypto modules are properly loaded. Please issue the following command:
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
**Note:** As you can see, there is already some crypto offload regarding DRBG-SHA-512. That is a good start ;D

**Note 2:** For your information, DRBG stands for Deterministic Random Bits Generator. It is an algorithm for generating a sequence of numbers whose properties approximate the properties of sequences of random numbers.

Your hands-on LAB environment is now properly setup!

## Pervasive Encryption - Enabling OpenSSL and OpenSSH to use the hardware acceleration support

This chapter describes how to use the cryptographic functions of the LinuxONE server to encrypt data in flight. This technique means that the data is encrypted and decrypted before and after it is transmitted. We will use OpenSSL, SCP and SFTP to demonstrate the encryption of data in flight. 
This chapter also shows how to customize the product to use the LinuxONE hardware encryption features. This chapter includes the following sections:
- Preparing to use openSSL
- Configuring OpenSSL
- Testing Hardware Crypto functions with OpenSSL
- Testing Hardware Crypto functions with SCP

### 1. Preparing to use OpenSSL
In the Linux system you use, OpenSSL is already installed, and the system is already enabled to use the cryptographic hardware of the LinuxONE server. We also loaded the cryptographic device drivers and the libica to use the crypto hardware. For the following steps, the following packages are required for encryption:
- openssl
- openssl-libs
- openssl-ibmca

During the installation of Ubuntu, the package openssl-ibmca was not automatically installed and needs to be installed manually. Please issue the following command:
```
root@crypt06:~# sudo apt-get install openssl-ibmca
```
Now all needed packages are successfully installed. At this moment only the default engine of OpenSSL is available. To check it, please issue the following command:
```
root@crypt06:~# openssl engine -c
(dynamic) Dynamic engine loading support
```
### 2. Configuring OpenSSL
To use the ibmca engine and to benefit from the Cryptographic hardware support, the configuration file of OpenSSL needs to be adjusted. To customize OpenSSL configuration to enable dynamic engine loading for ibmca, complete the following steps.
#### 2.1. Locate the OpenSSL configuration file, which in our Ubuntu 16.04.3 LTS distribution is in this subdirectory: 
```
root@crypt06:~# ls /usr/share/doc/openssl-ibmca/examples
```

#### 2.2. Make a backup copy of the configuration file
Locate the main configuration file of openssl. Its name is openssl.cnf. Please issue the following command:
```
root@crypt06:~# ls -la /etc/ssl/openssl.cnf
-rw-r--r-- 1 root root 10835 Dec  7 21:43 /etc/ssl/openssl.cnf
```
Make a backup copy of the configuration file. We will modify it later, so just in case, please issue the following command:
```
root@crypt06:~# cp -p /etc/ssl/openssl.cnf /etc/ssl/openssl.cnf.backup
```
Check one more time that everything is alright and secured. Please issue the following command:
```
root@crypt06:~# ls -al /etc/ssl/openssl.cnf*
-rw-r--r-- 1 root root 10835 Dec  7 21:43 /etc/ssl/openssl.cnf
-rw-r--r-- 1 root root 10835 Dec  7 21:43 /etc/ssl/openssl.cnf.backup
```

#### 2.3. Append the ibmca-related configuration lines to the OpenSSL configuration file
```
root@crypt06:~# tee -a /etc/ssl/openssl.cnf < /usr/share/doc/openssl-ibmca/examples/openssl.cnf.sample
#
# OpenSSL example configuration file. This file will load the IBMCA engine
# for all operations that the IBMCA engine implements for all apps that
# have OpenSSL config support compiled into them.
#
# Adding OpenSSL config support is as simple as adding the following line to
# the app:
#
# #define OPENSSL_LOAD_CONF	1
#
openssl_conf = openssl_def

[openssl_def]
engines = engine_section


[engine_section]
ibmca = ibmca_section


[ibmca_section]

# The openssl engine path for libibmca.so.
# Set the dynamic_path to where the libibmca.so engine
# resides on the system.
dynamic_path = /usr/lib/s390x-linux-gnu/openssl-1.0.0/engines/libibmca.so
engine_id = ibmca
init = 1

#
# The following ibmca algorithms will be enabled by these parameters
# to the default_algorithms line. Any combination of these is valid,
# with "ALL" denoting the same as all of them in a comma separated
# list.
#
# RSA
# - RSA encrypt, decrypt, sign and verify, key lengths 512-4096
#
# RAND
# - Hardware random number generation
#
# CIPHERS
# - DES-ECB, DES-CBC, DES-CFB, DES-OFB, DES-EDE3, DES-EDE3-CBC, DES-EDE3-CFB,
#   DES-EDE3-OFB, AES-128-ECB, AES-128-CBC, AES-128-CFB, AES-128-OFB,
#   AES-192-ECB, AES-192-CBC, AES-192-CFB, AES-192-OFB, AES-256-ECB,
#   AES-256-CBC, AES-256-CFB, AES-256-OFB symmetric crypto
#
# DIGESTS
# - SHA1, SHA256, SHA512 digests
#
default_algorithms = ALL
#default_algorithms = RAND,RSA,CIPHERS,DIGESTS
```

#### 2.4. Append the ibmca-related configuration lines to the OpenSSL configuration file
The reference to the ibmca section in the OpenSSL configuration file needs to be inserted. Therefore, insert the following line as show below at the line 10.
"openssl_conf = openssl_def"
```
[root@crypt06:~# vi /etc/ssl/openssl.cnf
#
# OpenSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#
# This definition stops the following lines choking if HOME isn't
# defined.
HOME = .
RANDFILE = $ENV::HOME/.rnd
openssl_conf = openssl_def #<== line inserted
# Extra OBJECT IDENTIFIER info:
#oid_file = $ENV::HOME/.oid
oid_section = new_oids
# To use this configuration file with the "-extfile" option of the
# "openssl x509" utility, name here the section containing the
# X.509v3 extensions to use:
```

### 3. Checking Hardware Crypto functions
Now that the customization of OpenSSL in done, test whether you can use the LinuxONE hardware cryptographic functions. First, let's check whether the dynamic engine loading support is enabled by default and the engine ibmca is available and used in the installation.
```
root@crypt06:~# openssl engine -c
(dynamic) Dynamic engine loading support
(ibmca) Ibmca hardware engine support
 [RAND, DES-ECB, DES-CBC, DES-OFB, DES-CFB, DES-EDE3, DES-EDE3-CBC, DES-EDE3-OFB, DES-EDE3-CFB, AES-128-ECB, AES-192-ECB, AES-256-ECB, AES-128-CBC, AES-192-CBC, AES-256-CBC, AES-128-OFB, AES-192-OFB, AES-256-OFB, AES-128-CFB, AES-192-CFB, AES-256-CFB, SHA1, SHA256, SHA512]
```

#### 3.1. Testing Pervasive encryption with OpenSSL
Now openSSL is properly configured. All crypto operation passing through openSSL will be hardware accelerated if possible. Let's test it in live. Firt of all, let's have a look of the hardware crypto offload status. Please issue the following command:
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
 DRBG-SHA-512 |             676          |                0
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
As you can see, the libica API already detect some crypto offload to the hardware.

Let's go deeper with some openSSL tests. Please issue the following command:
```
root@crypt06:~# openssl speed -evp aes-128-cbc 
Doing aes-128-cbc for 3s on 16 size blocks: 33394917 aes-128-cbc's in 2.83s
Doing aes-128-cbc for 3s on 64 size blocks: 30227460 aes-128-cbc's in 2.84s
Doing aes-128-cbc for 3s on 256 size blocks: 21680556 aes-128-cbc's in 2.87s
Doing aes-128-cbc for 3s on 1024 size blocks: 9606871 aes-128-cbc's in 2.91s
Doing aes-128-cbc for 3s on 8192 size blocks: 1772172 aes-128-cbc's in 2.89s
OpenSSL 1.0.2g  1 Mar 2016
built on: reproducible build, date unspecified
options:bn(64,64) rc4(8x,char) des(idx,cisc,16,int) aes(partial) blowfish(idx) 
compiler: cc -I. -I.. -I../include  -fPIC -DOPENSSL_PIC -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -DB_ENDIAN -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -Wl,-Bsymbolic-functions -Wl,-z,relro -Wa,--noexecstack -Wall -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DAES_ASM -DAES_CTR_ASM -DAES_XTS_ASM -DGHASH_ASM
The 'numbers' are in 1000s of bytes per second processed.
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes
aes-128-cbc     188805.18k   681182.20k  1933875.38k  3380562.17k  5023402.43k
```
The last line is quite interesting, it shows the encrypion bandwidth of one IFL - encrypting blocks of 8192 bytes of data over and over, as fast as possible. In this case, the observed throughput is roughly 5 GB/s (5023402.43/(1024\*1024)).

Let's test now the decryption capabilities. Please issue the following command:
```
root@crypt06:~# openssl speed -evp aes-128-cbc -decrypt
Doing aes-128-cbc for 3s on 16 size blocks: 33183167 aes-128-cbc's in 2.86s
Doing aes-128-cbc for 3s on 64 size blocks: 32184259 aes-128-cbc's in 2.87s
Doing aes-128-cbc for 3s on 256 size blocks: 27682462 aes-128-cbc's in 2.88s
Doing aes-128-cbc for 3s on 1024 size blocks: 16928435 aes-128-cbc's in 2.91s
Doing aes-128-cbc for 3s on 8192 size blocks: 5391831 aes-128-cbc's in 2.93s
OpenSSL 1.0.2g  1 Mar 2016
built on: reproducible build, date unspecified
options:bn(64,64) rc4(8x,char) des(idx,cisc,16,int) aes(partial) blowfish(idx) 
compiler: cc -I. -I.. -I../include  -fPIC -DOPENSSL_PIC -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -DB_ENDIAN -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -Wl,-Bsymbolic-functions -Wl,-z,relro -Wa,--noexecstack -Wall -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DAES_ASM -DAES_CTR_ASM -DAES_XTS_ASM -DGHASH_ASM
The 'numbers' are in 1000s of bytes per second processed.
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes
aes-128-cbc     185640.10k   717697.76k  2460663.29k  5956947.57k 15075044.22k
```
Same computation, as you can see the observed throuthput is 15 GB/s. That is normal because the CBC mode of operation associated with the AES encryption algorithm is faster in decryption that in encryption.

Let's check now how much crypto we offload to the hardware. Please issue the following command:
```
root@crypt06:~# icastats
 function     |          # hardware      |       # software
--------------+--------------------------+-------------------------
              |       ENC    CRYPT   DEC |        ENC    CRYPT   DEC
--------------+--------------------------+-------------------------
        SHA-1 |             236          |                0
      SHA-224 |               0          |                0
      SHA-256 |               0          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |            1014          |                0
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
      AES CBC |  96681976      115370154 |         0             0
      AES OFB |         0              0 |         0             0
      AES CFB |         0              0 |         0             0
      AES CTR |         0              0 |         0             0
     AES CMAC |         0              0 |         0             0
      AES XTS |         0              0 |         0             0
```
**Note:** We can clearly see here the crypto offload in encryption operations. 96681976 operations were offloaded to the CPACF.
**Note2:** We can clearly see here the crypto offload in decryption operations. 115370154 operations were offloaded to the CPACF.

#### 3.2. Testing Pervasive encryption with scp
The scp command allows you to copy files over ssh connections. This is pretty useful if you want to transport files between computers, for example to backup something. The scp command uses the ssh protocol and they are very much alike. However, there are some important differences.
The scp command can be used in three* ways: to copy from a (remote) server to your computer, to copy from your computer to a (remote) server, and to copy from a (remote) server to another (remote) server. In the third case, the data is transferred directly between the servers; your own computer will only tell the servers what to do. These options are very useful for a lot of things that require files to be transferred
Let's first clean the icastats monitoring. Please issue the following command:
```
root@crypt06:~# icastats -r
```
The previous command reset the icastats monitoring interface, and ease to interpret future result. 

To confirm the reset took place, please issue the following command:
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
 DRBG-SHA-512 |               0          |                0
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

Let's now create a 512MB file that we will use later for a secure file transfer. 

Please issue the following command and note that it will take less than a minute:
```
root@crypt06:~# dd if=/dev/urandom of=data.512M bs=64M count=8 iflag=fullblock
8+0 records in
8+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 45.9584 s, 11.7 MB/s
```
You just created a 512MB file made of random value. The file is named data.512MB.

You can confirm it issuing the following command:
```
root@crypt06:~# ls -hs
total 512M
512M data.512M
```
Now, let's see the hardware crypto offload with the SCP network protocol.

Please issue the following command, yes confirm if prompted, and enter your session password if prompted (azerty11):
```
root@crypt06:~# scp data.512M localhost:/dev/null
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:IRiJuwD8Z8NF77bRUfNIfcIYEbocWMVT6RVcZh+FChs.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
root@localhost's password: <----- azerty11

data.512M                                                         100%  512MB 170.7MB/s   00:03
```
As you can see, the defaut cipher suite transfered 512MB in 3 seconds, so a bandwidth of 170MB/s. Not so bad.

Issue the icastats command again to validate the hardware crypto offload:
```
root@crypt06:~# icastats
 function     |          # hardware      |       # software
--------------+--------------------------+-------------------------
              |       ENC    CRYPT   DEC |        ENC    CRYPT   DEC
--------------+--------------------------+-------------------------
        SHA-1 |             388          |                0
      SHA-224 |               0          |                0
      SHA-256 |              70          |                0
      SHA-384 |               0          |                0
      SHA-512 |               0          |                0
        GHASH |               0          |                0
        P_RNG |               0          |                0
 DRBG-SHA-512 |             338          |                0
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
**Note:** Data integrity operation with sha1 and sha256 were correcly reported. AES is accelerated by default and is not reported because not passing through libica device driver.

**Note2:** By default, scp doesn't use the best ciphers of IBM Z and LinuxONE. However, it is possible to specify it manually with -c option.

Let's compare the defaut cipher performance with an optimized cipher. Please issue the following command:
```
root@crypt06:~# scp -c aes256-ctr data.512M localhost:/dev/null
root@localhost's password: 
data.512M                                                           100%  512MB 256.0MB/s   00:02
```
As you can see, with 256MB/s, we increased the throughput by 50%. So, beware default settings. Make sure to use hardware accelerated ciphers by IBM Z and LinuxONE.

## Pervasive Encryption - Enabling dm-crypt to use the Hardware
The objective of this chapter, is to protect data at rest using dm-crypt in order to encrypt volumes. dm-crypt is very interresting, in that it helps to encrypt data without stopping running applications.

For the following, we will use LVM method to protect data at rest with dm-crypt at volume level. Objective will be to migrate data from an unencrypted volume to a (dm-crypt) encrypted volume. This is a 4 steps approach that doesn't required to reboot or to stop running application. 
4 steps includes the following:
* Step 1: Format a new encrypted volume with dm-crypt
* Step 2: Add dm-crypt based physical volume to volume group
* Step 3: Migrate data from non encrypted volume to encrypted volume
* Step 4: Remove unencrypted volume from the volume group

Let's do this for real now.

In your lab machine environnement there is an application running in a docker container (tomcat server). Please issue the following command:
```
root@crypt06:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
a96459e351d6        s390x/tomcat:jre9   "catalina.sh run"   2 days ago          Up 2 days           0.0.0.0:8080->8080/tcp   upbeat_kare
```

You can also confirm that the application is running using a web-browser and checking the url:
```
http://<your_lab_machine_ip>:8080/
```
![Image of still running tomcat application](https://github.com/guikarai/PE-LinuxONE/blob/master/images/tomcat-running.png)

Docker application and its data has been mounted in a volume group named vg01. This is the volume we will encryption without impacting running docker application. To assess the docker configuration on disk, please issue the following commands:
```
root@crypt06:~# dmsetup status
vg01-lv01: 0 20963328 linear 
```
First of all, the volume group vg01 is not encrypted.

```
root@crypt01:~# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
dasda          94:0    0  6.9G  0 disk 
`-dasda1       94:1    0  6.9G  0 part /
dasdb          94:4    0  1.4G  0 disk 
`-dasdb1       94:5    0  1.4G  0 part [SWAP]
dasdc          94:8    0   10G  0 disk 
`-dasdc1       94:9    0   10G  0 part 
  `-vg01-lv01 252:0    0   10G  0 lvm  /var/lib/docker
dasdd          94:12   0   10G  0 disk 
`-dasdd1       94:13   0   10G  0 part
```
Second, /var/lib/docker is mounted on the volume group vg01.

### 1. Installing cryptsetup
The cryptsetup feature provides an interface for configuring encryption on block devices (such as /home or swap partitions), using the Linux kernel device mapper target dm-crypt. It features integrated LUKS (Linux Unified Key Setup) support. LUKS standardizes the format of the encrypted disk, which allows different implementations, even from other operating systems, to access and decrypt the disk. 
LUKS adds metadata to the underlying block device, which contains information about the ciphers used and a default of eight key slots that hold an encrypted version of the master key used to decrypt the device. 
You can unlock the key slots by either providing a password on the command line or using a key file, which could, for example, be encrypted with gpg and stored on an NFS share.
First of all, let's install cryptsetup, you can issue the following command:
```
root@crypt06:~# sudo apt-get install cryptsetup
Reading package lists... Done
Building dependency tree       
Reading state information... Done
cryptsetup is already the newest version (2:1.6.6-5ubuntu2.1).
0 upgraded, 0 newly installed, 0 to remove and 106 not upgraded.
```

The dm-crypt feature supports various ciphers and hashing algorithms that you can select from the ones that are available in the Kernel and listed in the /proc/crypto procfs file. This also means that dm-crypt takes advantage of the unique hardware acceleration features of IBM Z that increase encryption and decryption speed.
Using the cryptsetup command, create a LUKS partition on the respective disk devices. For full disk encryption, use the AES xts hardware feature. We choose the AES-xts to achieve a security level of reasonable quality with the best encryption mode.

To confirm that is the best choice, you can issue the following command:
```
[root@crypt06:~# cryptsetup benchmark
# Tests are approximate using memory only (no storage IO).
PBKDF2-sha1       840205 iterations per second
PBKDF2-sha256     491827 iterations per second
PBKDF2-sha512     337379 iterations per second
PBKDF2-ripemd160  599871 iterations per second
PBKDF2-whirlpool  179796 iterations per second
#  Algorithm | Key |  Encryption |  Decryption
     aes-cbc   128b  2406.2 MiB/s  3821.1 MiB/s
 serpent-cbc   128b    75.6 MiB/s    89.4 MiB/s
 twofish-cbc   128b   147.0 MiB/s   175.2 MiB/s
     aes-cbc   256b  2196.9 MiB/s  3655.5 MiB/s
 serpent-cbc   256b    75.9 MiB/s    86.1 MiB/s
 twofish-cbc   256b   146.5 MiB/s   169.3 MiB/s
     aes-xts   256b  3330.6 MiB/s  3621.2 MiB/s
 serpent-xts   256b    75.8 MiB/s    85.5 MiB/s
 twofish-xts   256b   160.6 MiB/s   165.2 MiB/s
     aes-xts   512b  3770.6 MiB/s  3537.9 MiB/s
 serpent-xts   512b    77.9 MiB/s    86.7 MiB/s
 twofish-xts   512b   163.0 MiB/s   165.0 MiB/s

```
### 2. Using dm-crypt Volumes as LVM Physical Volumes
#### 2.1. Initial physical volume assessment
```
root@crypt06:~# pvs
  PV          VG   Fmt  Attr PSize  PFree 
  /dev/dasdc1      lvm2 ---  10.00g 10.00g
  /dev/dasdd1 vg01 lvm2 a--  10.00g     0
```

**Note:** There is one physical volume which is not yet part of any volume group. This is the disk we'll use with cryptsetup.
In this example, this device is /dev/dasdc1. Yours may be different.

#### 2.2. Initial volume group assessment
```
root@crypt06:~# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg01   1   1   0 wz--n- 10.00g    0
```

#### 2.3. Initial logical volume assessment
```
root@crypt06:~# lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv01 vg01 -wi-ao---- 10.00g
```

### 3.1 Step 1 - Formatting and encrypting a new volume
In the following step, we will format and encrypt an existing volume.
![Step1](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step1.png)
For the follwing, we will use a passphrase to generate an encryption key. Be sure to remember the entered passphrase. You can simply use "cryptlab" to make it simple. The following command will create a cryptographic device mapper device in LUKS encryption mode:

```
root@crypt06:~# cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/dasdc1

WARNING!
========
This will overwrite data on /dev/dasdc1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase:  <--- cryptlab
Verify passphrase: <--- cryptlab
```

Next, you have to open the volume onto the device mapper. This is the stage at which you will be prompted for your passphrase. You can choose the name that you want your partition mapped under. We proposed to you "dockercrypt".
```
root@crypt06:~# cryptsetup luksOpen /dev/dasdc1 dockercrypt
Enter passphrase for /dev/dasdc1: 
```

### 3.2 Step 2 - Add dm-crypt based physical volume to volume group
In this second step, we will add the encrypted volume into the existing volume group. Both an encrypted volume and unencrypted volume will be part of the same volume groupe.
![Step2](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step2.png)

Now we will use the device as phisical volume. Please issue the following command:
```
root@crypt06:~# pvcreate /dev/mapper/dockercrypt 
  Physical volume "/dev/mapper/dockercrypt" successfully created.
```

Now we will add in the volume group the new phisical device /dev/mapper/dockercrypt. Please issue the following command:
```
root@crypt06:~# vgextend vg01 /dev/mapper/dockercrypt 
  Volume group "vg01" successfully extended
```

Let's check the new volume group status. As you can see, there is now 2 physical volume in the volume group.
```
root@crypt06:~# vgs
  VG    #PV #LV #SN Attr   VSize  VFree  
  vg01   2   1   0 wz--n- 49.99g <25.00g
```

Checking physical volume, we can see that the encrypted new one is there aside from the initial one. Both part of the same volume group vg01.
```
root@crypt06:~# pvs
  PV                      VG    Fmt  Attr PSize   PFree  
  /dev/mapper/dockercrypt vg01  lvm2 a--  10.00g  10.00g
  /dev/dasdd1             vg01  lvm2 a--  10.00g      0 
```

You can use sudo dmsetup status to check if there are any LUKS-encrypted partitions. The output should look something like:
```
root@crypt06:~# dmsetup status
dockercrypt: 0 20967872 crypt 
vg01-lv01: 0 20963328 linear 
```
The line marked "crypt" shows that sda5 has been encrypted. You can see which filesystems are on that via the lvm tools.


### 3.3 Step 3 - Migrate data from non encrypted volume to encrypted volume
In this third step, we will migrate unencrypted data in the uncrypted physical volume to the encrypted physical volume. This operation thanks to the **pvmove** switch update from the source PV to the destination PV once completed. This command doesn't halt running application. This operation can take some time. 
![Step3](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step3.png)

Please issue to following command to proceede:
```
root@crypt06:~# pvmove /dev/dasdd1 /dev/mapper/dockercrypt
  /dev/dasdd1: Moved: 0.00%
  /dev/dasdd1: Moved: 4.83%
  /dev/dasdd1: Moved: 9.24%
  /dev/dasdd1: Moved: 13.13%
  /dev/dasdd1: Moved: 17.16%
  /dev/dasdd1: Moved: 22.02%
  /dev/dasdd1: Moved: 27.55%
  /dev/dasdd1: Moved: 33.08%
  /dev/dasdd1: Moved: 36.91%
  /dev/dasdd1: Moved: 40.98%
  /dev/dasdd1: Moved: 45.46%
  /dev/dasdd1: Moved: 48.55%
  /dev/dasdd1: Moved: 50.91%
  /dev/dasdd1: Moved: 53.52%
  /dev/dasdd1: Moved: 57.02%
  /dev/dasdd1: Moved: 59.99%
  /dev/dasdd1: Moved: 63.03%
  /dev/dasdd1: Moved: 66.34%
  /dev/dasdd1: Moved: 69.78%
  /dev/dasdd1: Moved: 73.37%
  /dev/dasdd1: Moved: 76.53%
  /dev/dasdd1: Moved: 79.93%
  /dev/dasdd1: Moved: 83.43%
  /dev/dasdd1: Moved: 86.83%
  /dev/dasdd1: Moved: 90.47%
  /dev/dasdd1: Moved: 95.17%
  /dev/dasdd1: Moved: 98.56%
  /dev/dasdd1: Moved: 100.00%
```

### 3.4 Step 4 - Remove unencrypted volume from the volume group
Fourth and last step. It is time to remove from the volume group, the volume that is unencrypted, we don't need it anymore.
![Step4](https://github.com/guikarai/PE-LinuxONE/blob/master/images/step4.png)

Do do so, we will use the **vgreduce**. Please issue the following command:
```
root@crypt06:~# vgreduce vg01 /dev/dasdd1
  Removed "/dev/dasdd1" from volume group "vg01"
```

Let's check the new configuration of LV, VG and PV: Please issue the following command:
```
root@crypt06:~# lvs
  LV   VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv01 vg01  -wi-ao---- 10.00g                                                    
```

```
root@crypt06:~# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg01   1   1   0 wz--n- 10.00g    0
 
```

```
root@crypt06:~# pvs
  PV                      VG   Fmt  Attr PSize  PFree 
  /dev/dasdc1                  lvm2 ---  10.00g 10.00g
  /dev/dasdd1                  lvm2 ---  10.00g 10.00g
  /dev/mapper/dockercrypt vg01 lvm2 a--  10.00g     0 
```

Now, application run 100% on a encrypted volume. Let's check if application is still running. Please issue the following command:
```
root@crypt06:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
a96459e351d6        s390x/tomcat:jre9   "catalina.sh run"   2 days ago          Up 2 days           0.0.0.0:8080->8080/tcp   upbeat_kare
```

You can also confirm that the application is running using a web-browser and checking the url:
```
http://<your_lab_machine_ip>:8080/
```
If you see the following, you did it!
![Image of still running tomcat application](https://github.com/guikarai/PE-LinuxONE/blob/master/images/tomcat-running.png)

To guaranty you that docker runs on the encrypted devices, please issue the following command:
```
root@crypt06:~# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
dasda            94:0    0  6.9G  0 disk  
`-dasda1         94:1    0  6.9G  0 part  /
dasdb            94:4    0  1.4G  0 disk  
`-dasdb1         94:5    0  1.4G  0 part  [SWAP]
dasdc            94:8    0   10G  0 disk  
`-dasdc1         94:9    0   10G  0 part  
  `-dockercrypt 252:1    0   10G  0 crypt 
    `-vg01-lv01 252:0    0   10G  0 lvm   /var/lib/docker
dasdd            94:12   0   10G  0 disk  
`-dasdd1         94:13   0   10G  0 part  
```

To be sure that there is a prompt after after a reboot, please create /etc/crypttab with the following content:
```
root@crypt06:~# vi /etc/crypttab
dockercrypt /dev/disk/by-path/ccw-0.0.0201-part1 none
```

You just finished the Pervasive encryption LAB. Thank you, and see you soon.
