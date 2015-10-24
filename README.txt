OpenVPN 3
-----------

OpenVPN 3 is a C++ class library that implements the functionality
of an OpenVPN client, and is protocol-compatible with the OpenVPN
2.x branch.

OpenVPN includes a small client wrapper (cli) that links in with
the library and provides minimal command line functionality.

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
