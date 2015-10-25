OpenVPN 3
-----------

OpenVPN 3 is a C++ class library that implements the functionality
of an OpenVPN client, and is protocol-compatible with the OpenVPN
2.x branch.

OpenVPN includes a small client wrapper (cli) that links in with
the library and provides minimal command line functionality.

===================================================================
===================================================================
===================================================================
Building OpenVPN 3 for Android (added by jmaurice)

# create new VM
install ubuntu-12.04.5-server-i386.iso

# install OS deps

rm -rf /var/lib/apt/lists/*
vi /etc/apt/sources.list - uncomment partner and extras
apt-get update
apt-get upgrade
apt-get install openjdk-7-jdk gcc g++ cmake binutils zsh vim git unzip zip rar unrar make autoconf automake libtool swig

# download android deps

mkdir ~/src/android/

download deps into ~/src/android/
  android-sdk_r24.3.4-linux.tgz into ~/src/android
  android-ndk-r10e-linux-x86.bin into ~/src/android

download into ~/Downloads
  boost_1_55_0.tar.gz
  lz4-r112.tar.gz
  lzo.tar.gz
  openssl-1.0.1p.tar.gz
  polarssl-1.3.6-gpl.tgz
  snappy-1.1.1.tar.gz

# extract android deps

extract android-sdk_r24.3.4-linux.tgz into ~/src/android
extract android-ndk-r10e-linux-x86.bin into ~/src/android

# install android deps

$HOME/src/android/android-sdk-linux/tools/android update sdk --no-ui --all

# build prep
clone openvpn3 repo
export O3=$HOME/openvpn3
export DEP_DIR=$HOME/src/android

# build toolchain & deps

scripts/android/build-toolchain
scripts/android/build-all

# build libovpn.so

cd javacli
cp /usr/lib/jvm/java-7-openjdk-i386/include/jni.h ~/src/android/tc/sysroot/usr/include/
cp /usr/lib/jvm/java-7-openjdk-i386/include/jni_md.h ~/src/android/tc/sysroot/usr/include/
cp /usr/lib/jvm/java-7-openjdk-i386/include/jawt_md.h ~/src/android/tc/sysroot/usr/include/
./build-android

# save build output

-rwxr-xr-x 1 jmaurice jmaurice 1.1M Oct 25 00:41 build/libs/armeabi-v7a/libovpncli.so
-rwxr-xr-x 1 jmaurice jmaurice 1.4M Oct 25 00:40 build/libs/armeabi/libovpncli.so

===================================================================
===================================================================
===================================================================

Building OpenVPN 3 on Mac OS X
------------------------------

OpenVPN 3 should be built in a non-root Mac OS X account.
Make sure that Xcode is installed with optional command-line tools.
(These instructions have been tested with Xcode 5.0.2).

Create a directory ~/src and ~/src/mac.  This can be done with the
command:

  mkdir -p ~/src/mac

Expand the OpenVPN 3 tarball:

 cd ~/src
 tar xf openvpn3.tar.gz

Export the shell variable O3 to point to the OpenVPN 3 top level
directory:

 export O3=~/src/openvpn3

Download source tarballs (.tar.gz or .tgz) for these dependency
libraries into ~/Downloads

See the file ~/src/openvpn3/lib-versions for the expected
version numbers of each dependency.  If you want to use a different
version of the library than listed here, you can edit this file.

1. Boost -- http://www.boost.org/
2. PolarSSL (1.3.4 or higher) -- https://polarssl.org/
3. OpenSSL (1.0.1) -- http://www.openssl.org/
4. Snappy -- https://code.google.com/p/snappy/
5. LZ4 -- https://code.google.com/p/lz4/

Note that while LZO is listed in lib-versions, it is
not required for Mac builds.

OpenSSL is required, however OpenVPN 3 for Mac doesn't use OpenSSL
in the standard way.  Instead, it cherry-picks some ASM-optimized
crypto and hash algorithms from OpenSSL to speed up PolarSSL's
low-level crypto processing, and assembles them into a library
called libminicrypto.a.

Build the dependencies:

  OSX_ONLY=1 $O3/scripts/mac/build-all

Now build the OpenVPN 3 client executable:

  cd $O3
  . vars-osx
  . setpath
  cd test/ovpncli
  GCC_EXTRA="-std=c++11" STRIP=1 PSSL=1 MINI=1 SNAP=1 LZ4=1 build cli

This will build the OpenVPN 3 client library with a small client
wrapper (cli).  It will also statically link in all external
dependencies (Boost, PolarSSL, libminicrypto (derived from OpenSSL),
LZ4, and Snappy), so "cli" may be distributed to other Macs and
will run as a standalone executable.  It's best to build OpenVPN 3
in C++11 mode to allow for the use of optimized move constructors.
But keep in mind that OpenVPN 3 strictly follows the C++ 2003
standard (not C++ 2011), so building with -std=c++11 is optional
for optimization purposes.

These build scripts will create a "fat" Mac OS X executable with
support for both x86_x64 and i386 architectures, with a minimum
deployment target of 10.6.x.  The Mac OS X tuntap driver is not
required, as OpenVPN 3 can use the integrated utun interface if
available.

To view the client wrapper options:

  ./cli -h

To connect:

  ./cli client.ovpn
