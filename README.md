# WolfSSL 4.7.0 with Postquantum Algorithms for PC

## Description
This is a tweeked version of WolfSSL that enables the use of Postquantum algorithms in TLS 1.3 key exchange and authentication methods, designed to run on PC's. The version we are using is [WolfSSL 4.7.0](https://github.com/wolfSSL/wolfssl/releases/tag/v4.7.0-stable) and the postquantum algorithms are from [PQClean](https://github.com/PQClean/PQClean). The postquantum algorithms that are supported are:


### Key Encapsualtion Mechanisms (KEMs)

- [x] CRYSTALS-Kyber

- [x] Saber

- [x] NTRU

- [x] SIKE

- [x] BIKE

- [x] HQC

- [x] NTRU Lprime

- [x] FrodoKEM

### Digital Signature Algorithms

- [x] CRYSTALS-Dilithium

- [x] Falcon

- [x] SPHINCS+

- [x] Picnic3

## System Requirements

This version is targeting theoretically any Linux system and it requires: `git` `autoconf` `automake` `libtool` and possibly other, more common packages like `gcc` or `make`. 

Install the requirements:
```
sudo apt update
sudo apt install make autoconf automake libtool git
```

The only thing on this project that is system specific is the **PQClean crypto libraries**. 

Below we have included a script that takes care of the installation of the necessary libraries from PQClean.


## Installation

### Ubuntu (Debian based)

1. Clone this repository on your local system:

```
git clone https://gitlab.com/g_tasop/pq-wolfssl-for-pc
```

2. Clone the PQClean library on the same directory as this project:

```
git clone https://github.com/PQClean/PQClean.git
git reset --hard e11209a9f1808e23132b0559c795c2b716fb0d3d
```

3. Cd in the clone repository and run the auto generation script:

```
cd pq-wolfssl-for-pc
./autogen.sh
```

autogen.sh will take care of everything your systems needs for the installation _**and also**_ will build the PQClean libraries for your system and copy them on the wolfSSL project.

4. Now run the configure script with only TLS 1.3 enabled and the creation of libraries disabled:

```
./configure --enable-tls13 --disable-tlsv12 --disable-oldtls --disable-shared --disable-static
```
(For debug purposes you should pass the `--enable-debug` option too)

5. Before we build the project we must edit the `config.h` file and add these lines on the top:
```
/* ----------------------------------------------------------------------------------*/
/* Post-quantum TLS 1.3 Benchmark */
/* ----------------------------------------------------------------------------------*/


	#define HAVE_POSTQUANTUM	// Enables post-quantum in TLS 1.3
	/* ------------------------------------------------------------------------------*/
	#define HAVE_DILITHIUM		// 		choose
//	#define HAVE_FALCON			// 	authentication
//	#define HAVE_PICNIC			// 		method
//	#define HAVE_SPHINCS		//
	/* ------------------------------------------------------------------------------*/
	#define HAVE_KYBER			//
//	#define HAVE_SABER			//
//	#define HAVE_FRODO			//		choose
//	#define HAVE_BIKE			//		key
//	#define HAVE_SIKE			//	 	exchange
//	#define HAVE_HQC			//		method
//	#define HAVE_NTRULPR		//
//	#define HAVE_NTRU_PQM4		//
	/* ------------------------------------------------------------------------------*/
//	#define HAVE_TRADITIONAL	// Enable to have TLS with RSA sign (regardless of HAVE_POSTQUANTUM)
//	#define MY_HAVE_ECC			// Enable to have TLS with ECC sign
	/* ------------------------------------------------------------------------------*/
	#define HAVE_MUTUAL_AUTH	// Enable to have mutual authentication


 /*
  * To control the *security levels* of each PQ algorithm you must go to the
  * corresponding security file. For example for Kyber:
  *
  * Go to <wolfcrypt/postquantum/kyber/kyber_security.h> and enable one of
  * the defines for the security levels.
  *
  * For Dilithium the path is <wolfcrypt/postquantum/dilithium/dilithium_security.h>
  * and so on...
  *
  */



```
6. Now we are ready to build the whole project:

```
make
```
** **NOTICE:** ** there is a problem with newer distibutions (i.e Ubuntu 21.10) that use gcc-11 by default. Fallback to a previous version, i.e gcc-10, by running:

```
make CC=gcc-10
```

Optionally we can install the library on the system for using PQ WolfSSL on programs with a simple library call (-lwolfssl).

**_(Not recommended yet)_**

```
sudo make install
```

Additionally, one can pass harware optimization flags to the compiler to make use of _Intel assembly_ and _aesni_ options. Keep in mind that some things may **NOT** be working properly with these flags. Use it on your own risk.

```
./configure --enable-tls13 --disable-tlsv12 --disable-oldtls --disable-shared --disable-static --enable-intelasm --enable-aesni --enable-sp --enable-sp-asm
```

### Windows

Maybe you can import this project to EclipseCDT (or maybe another IDE) as a Makefile project or even as an Autotools project and do the Ubuntu installation steps 3-6 from the IDE. Unfortunately steps 1-2 must be done manually.

(not tested)

## Test

To test the correct functionality of the project we can run some examples. These examples use the "examples/benchmark/tls_bench" program that offers a lot of functionality. You can check it by running:

```
./examples/benchmark/tls_bench -?
```

### Example 1

We can run a local TLS benchmark that creates two threads, one for the server and one for the client, and exchange messages using sockets. For sockets to work we must find our machine's local IP and give it as input to the benchmark. 

For example if our local IP is "192.168.1.14" and we want each test to run for 2 seconds, we can do the following:

Being on the top level directory run:
```
./examples/benchmark/tls_bench -h "192.168.1.14" -t 2
```

### Example 2

We can open two terminals and run a TLS server and a TLS client to connect to each other using the local network.

One the first terminal run the server:
```
./examples/benchmark/tls_bench -s -t 2
```

and on the second one, the client:
```
./examples/benchmark/tls_bench -ch "192.168.1.14" -t 2
```

### Expected results

The results should be something like this:

```
Running TLS Benchmarks...
TLS key exchange method: 		Kyber1024
TLS authentication method: 		Dilithium3

Side    Cipher                             Total Bytes  Num Conns      Rx ms      Tx ms    Rx MB/s    Tx MB/s   Connect Total ms   Connect Avg ms
Server  TLS13-AES128-GCM-SHA256                5251072        642    142.348     72.300     17.590     34.632           1705.740            2.657
Client  TLS13-AES128-GCM-SHA256                5251072        642    194.491     73.560     12.874     34.039           1682.280            2.620
Side    Cipher                             Total Bytes  Num Conns      Rx ms      Tx ms    Rx MB/s    Tx MB/s   Connect Total ms   Connect Avg ms
Server  TLS13-AES256-GCM-SHA384                5185536        634    155.804     78.679     15.870     31.427           1683.176            2.655
Client  TLS13-AES256-GCM-SHA384                5185536        634    212.076     79.194     11.659     31.223           1660.017            2.618
Side    Cipher                             Total Bytes  Num Conns      Rx ms      Tx ms    Rx MB/s    Tx MB/s   Connect Total ms   Connect Avg ms
Server  TLS13-CHACHA20-POLY1305-SHA256         6012928        735     88.614     46.209     32.356     62.048           1790.655            2.436
Client  TLS13-CHACHA20-POLY1305-SHA256         6012928        735    136.263     45.822     21.042     62.572           1765.171            2.402
```

### Troubleshooting

If the benchmark shows an error like this:

```
ERROR: failed to connect
Client Error: -1 (unknown error number)
```

try re-running the benchmark OR try debugging it with an IDE, if `--enable-debug` flag is passed.

## Usage

To use this project to connect to a remote board (i.e a Nucleo Board) you should do:

1. Set up a server to run on the local machine:

```
./examples/benchmark/tls_bench -sv -t 10 -l "TLS13-AES256-GCM-SHA384"
```

This will run a local server on verbose mode (-sv), set the benchmark time to 10 seconds (-t 10) and use only the pecified cipher suite (-l "TLS13-AES256-GCM-SHA384").

2. Get the internal IP if it's a local network or the external IP if it's connected to the Internet and use this [project](https://gitlab.com/g_tasop/pq-wolfssl-for-embedded) to setup and run the client from the board targeting that IP 

OR

1. Set up a server to run on the remote machine and get its IP. For example, the IP of the remote machine could be "192.168.1.14".

4. Run the client with the IP as an argument:

```
./examples/benchmark/tls_bench -cv -t 10 -l "TLS13-AES256-GCM-SHA384" -h "192.168.2.55"
```

This will run a TLS 1.3 client in verbose mode (-cv), set the benchmark time to 10 seconds (-t 10), use only the pecified cipher suite (-l "TLS13-AES256-GCM-SHA384") and connect to the host with the specified IP (-h "192.168.2.55")

## Tested

This project's build and usage has been tested successfully on the following platforms:

- [x] Dell Inspiron 14 7000, running Ubuntu 20.04 with a 11th Gen Intel® Core™ i7-1165G7 @ 2.80GHz × 8 processor.

- [x] Raspberry Pi 4 Model B, running Raspberry Pi OS with a 1.5 GHz 64-bit quad core ARM Cortex-A72 processor.

## Extra info

If you want to test the performance on **traditional** TLS 1.3 you only need to:

1. Comment out the Post-quantum defines that we added in file `wolfssl/wolfSSL.I-CUBE-wolfSSL_conf.h`.

2. Uncomment `#define HAVE_TRADITIONAL`.

3. Compile and run. 

The default algorithms that WolfSSL 4.7.0 uses are: 

- ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) with curve **SECP256R1** for key exchange.

- RSA-2048 for digital signatures.

To override them and use ECC in authentication you have to:

1.  Uncomment `#define MY_HAVE_ECC`.

2. Compile and run.

Now the traditional algorithms that are used are:

- ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) with curve **SECP256R1** for key exchange.

- ECDSA (Elliptic Curve DSA) with curve **SECP256R1** for authentication.



## License
For all the licenses of the components of this project, see file ``Third-party-licenses.txt`.

(the original README follows)

---

---


<a href="https://repology.org/project/wolfssl/versions">
    <img src="https://repology.org/badge/vertical-allrepos/wolfssl.svg" alt="Packaging status" align="right">
</a>

# wolfSSL Embedded SSL/TLS Library

The [wolfSSL embedded SSL library](https://www.wolfssl.com/products/wolfssl/) 
(formerly CyaSSL) is a lightweight SSL/TLS library written in ANSI C and
targeted for embedded, RTOS, and resource-constrained environments - primarily
because of its small size, speed, and feature set.  It is commonly used in
standard operating environments as well because of its royalty-free pricing
and excellent cross platform support. wolfSSL supports industry standards up
to the current [TLS 1.3](https://www.wolfssl.com/tls13) and DTLS 1.2, is up to
20 times smaller than OpenSSL, and offers progressive ciphers such as ChaCha20,
Curve25519, NTRU, and Blake2b. User benchmarking and feedback reports
dramatically better performance when using wolfSSL over OpenSSL.

wolfSSL is powered by the wolfCrypt cryptography library. Two versions of
wolfCrypt have been FIPS 140-2 validated (Certificate #2425 and
certificate #3389). FIPS 140-3 validation is in progress. For additional
information, visit the [wolfCrypt FIPS FAQ](https://www.wolfssl.com/license/fips/)
or contact fips@wolfssl.com.

## Why Choose wolfSSL?

There are many reasons to choose wolfSSL as your embedded, desktop, mobile, or
enterprise SSL/TLS solution. Some of the top reasons include size (typical
footprint sizes range from 20-100 kB), support for the newest standards
(SSL 3.0, TLS 1.0, TLS 1.1, TLS 1.2, TLS 1.3, DTLS 1.0, and DTLS 1.2), current
and progressive cipher support (including stream ciphers), multi-platform,
royalty free, and an OpenSSL compatibility API to ease porting into existing
applications which have previously used the OpenSSL package. For a complete
feature list, see [Chapter 4](https://www.wolfssl.com/docs/wolfssl-manual/ch4/)
of the wolfSSL manual.

## Notes, Please Read

**Note 1)**
wolfSSL as of 3.6.6 no longer enables SSLv3 by default.  wolfSSL also no longer
supports static key cipher suites with PSK, RSA, or ECDH. This means if you
plan to use TLS cipher suites you must enable DH (DH is on by default), or
enable ECC (ECC is on by default), or you must enable static key cipher suites
with one or more of the following defines:

    WOLFSSL_STATIC_DH
    WOLFSSL_STATIC_RSA
    WOLFSSL_STATIC_PSK

Though static key cipher suites are deprecated and will be removed from future
versions of TLS.  They also lower your security by removing PFS.  Since current
NTRU suites available do not use ephemeral keys, ```WOLFSSL_STATIC_RSA``` needs
to be used in order to build with NTRU suites.

When compiling ssl.c, wolfSSL will now issue a compiler error if no cipher
suites are available. You can remove this error by defining
```WOLFSSL_ALLOW_NO_SUITES``` in the event that you desire that, i.e., you're
not using TLS cipher suites.

**Note 2)**
wolfSSL takes a different approach to certificate verification than OpenSSL
does. The default policy for the client is to verify the server, this means
that if you don't load CAs to verify the server you'll get a connect error,
no signer error to confirm failure (-188).

If you want to mimic OpenSSL behavior of having SSL\_connect succeed even if
verifying the server fails and reducing security you can do this by calling:

    wolfSSL_CTX_set_verify(ctx, SSL_VERIFY_NONE, 0);

before calling wolfSSL\_new();. Though it's not recommended.

**Note 3)**
The enum values SHA, SHA256, SHA384, SHA512 are no longer available when
wolfSSL is built with --enable-opensslextra (```OPENSSL_EXTRA```) or with the
macro ```NO_OLD_SHA_NAMES```. These names get mapped to the OpenSSL API for a
single call hash function. Instead the name WC_SHA, WC_SHA256, WC_SHA384 and
WC_SHA512 should be used for the enum name.

# wolfSSL Release 4.7.0 (February 16, 2021)
Release 4.7.0 of wolfSSL embedded TLS has bug fixes and new features including:

### New Feature Additions
* Compatibility Layer expansion SSL_get_verify_mode, X509_VERIFY_PARAM API, X509_STORE_CTX API added
* WOLFSSL_PSK_IDENTITY_ALERT macro added for enabling a subset of TLS alerts
* Function wolfSSL_CTX_NoTicketTLSv12 added to enable turning off session tickets with TLS 1.2 while keeping TLS 1.3 session tickets available
* Implement RFC 5705: Keying Material Exporters for TLS
* Added --enable-reproducible-build flag for making more deterministic library outputs to assist debugging

### Fixes
* Fix to free mutex when cert manager is free’d
* Compatibility layer EVP function to return the correct block size and type
* DTLS secure renegotiation fixes including resetting timeout and retransmit on duplicate HelloRequest
* Fix for edge case with shrink buffer and secure renegotiation
* Compile fix for type used with curve448 and PPC64
* Fixes for SP math all with PPC64 and other embedded compilers
* SP math all fix when performing montgomery reduction on one word modulus
* Fixes to SP math all to better support digit size of 8-bit
* Fix for results of edge case with SP integer square operation
* Stop non-ct mod inv from using register x29 with SP ARM64 build
* Fix edge case when generating z value of ECC with SP code
* Fixes for PKCS7 with crypto callback (devId) with RSA and RNG
* Fix for compiling builds with RSA verify and public only
* Fix for PKCS11 not properly exporting the public key due to a missing key type field
* Call certificate callback with certificate depth issues
* Fix for out-of-bounds read in TLSX_CSR_Parse()
* Fix incorrect AES-GCM tag generation in the EVP layer
* Fix for out of bounds write with SP math all enabled and an edge case of calling sp_tohex on the result of sp_mont_norm
* Fix for parameter check in sp_rand_prime to handle 0 length values
* Fix for edge case of failing malloc resulting in an out of bounds write with SHA256/SHA512 when small stack is enabled


### Improvements/Optimizations
* Added --enable-wolftpm option for easily building wolfSSL to be used with wolfTPM
* DTLS macro WOLFSSL_DTLS_RESEND_ONLY_TIMEOUT added for resending flight only after a timeout
* Update linux kernel module to use kvmalloc and kvfree
* Add user settings option to cmake build
* Added support for AES GCM session ticket encryption
* Thread protection for global RNG used by wolfSSL_RAND_bytes function calls
* Sanity check on FIPs configure flag used against the version of FIPs bundle
* --enable-aesgcm=table now is compatible with --enable-linuxkm
* Increase output buffer size that wolfSSL_RAND_bytes can handle
* Out of directory builds resolved, wolfSSL can now be built in a separate directory than the root wolfssl directory

### Vulnerabilities
* [HIGH] CVE-2021-3336: In earlier versions of wolfSSL there exists a potential man in the middle attack on TLS 1.3 clients. Malicious attackers with a privileged network position can impersonate TLS 1.3 servers and bypass authentication. Users that have applications with client side code and have TLS 1.3 turned on, should update to the latest version of wolfSSL. Users that do not have TLS 1.3 turned on, or that are server side only, are NOT affected by this report. For the code change see https://github.com/wolfSSL/wolfssl/pull/3676.
* [LOW] In the case of using custom ECC curves there is the potential for a crafted compressed ECC key that has a custom prime value to cause a hang when imported. This only affects applications that are loading in ECC keys with wolfSSL builds that have compressed ECC keys and custom ECC curves enabled.
* [LOW] With TLS 1.3 authenticated-only ciphers a section of the server hello could contain 16 bytes of uninitialized data when sent to the connected peer. This affects only a specific build of wolfSSL with TLS 1.3 early data enabled and using authenticated-only ciphers with TLS 1.3.


For additional vulnerability information visit the vulnerability page at
https://www.wolfssl.com/docs/security-vulnerabilities/

See INSTALL file for build instructions.
More info can be found on-line at https://wolfssl.com/wolfSSL/Docs.html



# Resources

[wolfSSL Website](https://www.wolfssl.com/)

[wolfSSL Wiki](https://github.com/wolfSSL/wolfssl/wiki)

[FIPS 140-2/140-3 FAQ](https://wolfssl.com/license/fips)

[wolfSSL Documentation](https://wolfssl.com/wolfSSL/Docs.html)

[wolfSSL Manual](https://wolfssl.com/wolfSSL/Docs-wolfssl-manual-toc.html)

[wolfSSL API Reference](https://wolfssl.com/wolfSSL/Docs-wolfssl-manual-17-wolfssl-api-reference.html)

[wolfCrypt API Reference](https://wolfssl.com/wolfSSL/Docs-wolfssl-manual-18-wolfcrypt-api-reference.html)

[TLS 1.3](https://www.wolfssl.com/docs/tls13/)

[wolfSSL Vulnerabilities](https://www.wolfssl.com/docs/security-vulnerabilities/)
