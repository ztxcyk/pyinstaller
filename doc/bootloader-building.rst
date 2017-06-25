.. _building the bootloader:

Building the Bootloader
=========================

PyInstaller comes with pre-compiled bootloaders for some platforms in
the ``bootloader`` folder of the distribution folder.
When there is no pre-compiled bootloader, the pip_ setup will attempt to build one.

If there is no precompiled bootloader for your platform,
or if you want to modify the |bootloader| source,
you need to build the |bootloader|.
To do this,

* ``cd`` into the distribution folder.
* ``cd bootloader``.
* Make the bootloader with: ``python ./waf distclean all``.

This will produce the |bootloader| executables for your current platform
(operating-system and word-size):

* :file:`../PyInstaller/bootloader/{OS_ARCH}/run`,
* :file:`../PyInstaller/bootloader/{OS_ARCH}/run_d`,
* :file:`../PyInstaller/bootloader/{OS_ARCH}/runw` (MacOS and Windows only), and
* :file:`../PyInstaller/bootloader/{OS_ARCH}/runw_d` (MacOS and Windows only).

(Of course, for Windows these files will have the ``.exe`` extension.)

The bootloaders architecture defaults to the machine's one, but can be changed
using the ``--target-arch=`` option – given the appropriate compiler and
development files are installed. E.g. to build a 32-bit bootloader on a 64-bit
machine, run::

  python ./waf all --target-arch=32bit


If this reports an error, read the detailed notes that follow,
then ask for technical help.

Supported platforms are
* GNU/Linux (using gcc)
* Windows (using Visual C++ or MinGW's gcc)
* Mac OX X (using clang)

Contributed platforms are
* AIX (using gcc or xlc)
* HP-UX  (using gcc or xlc)
* Solaris

For more information about cross-building please read on
and mind the section about the Vagrantfile.


Building for GNU/Linux
========================

Development tools
----------------------

On Debian/Ubuntu-like systems, you can run the following to
install everything required::

    sudo apt-get install build-essential

On Fedora/RHEL and derivates, you can run the following::

    sudo yum groupinstall "Development Tools"


Building Linux Standard Base (LSB) compliant binaries
---------------------------------------------------------

By default, the bootloaders on Linux are ”normal“, non-LSB binaries, which
should be fine for all GNU/Linux distributions.

If for some reason you want to build Linux Standard Base (LSB) compliant
binaries [*]_, you can do so by specifying ``--lsb`` on the waf command line,
as follows::

       python ./waf distclean all --lsb

LSB version 4.0 is required for successfull building of |bootloader|. Please
refer to ``python ./waf --help`` for further options related to LSB building.

.. [*] Linux Standard Base (LSB) is a set of open standards that should
       increase compatibility among Linux distributions. Unfortunalty it is
       not widely adopted and both Debian and Ubuntu dropped support for LSB
       in autumn 2015. Thus |PyInstaller| bootloader are no longer provided
       as LSB binary.


Building for Mac OS X
========================

On Mac OS X you can get `clang` by installing Xcode_. It is a suite of tools
for developing software for Mac OS X. It can be also installed from your
Mac OS X Install DVD. It is not necessary to install the version 4 of Xcode.


Cross-Building for Mac OS X
-----------------------------------

For cross-compiling for OS X you need, the Clang/LLVM compiler, the the
`cctools` (ld, lipo, …), and the OSX SDK. Clang/LLVM is a cross compiler by
default and is available on nearly every GNU/Linux distribution, so you just
need a proper port of the cctools and the OS X SDK.

This is easy and needs to be done only once and the result can be transferred
to you build host. The build host can then be a normal (somewhat current)
GNU/Linux system. [*]_

.. [*] Please keep in mind that to avoid problems, the host you are using for
       the preparation steps should have the same architecture (and possible
       the same GNU/Linux distribution version) as the build host.

Preparation: Get SDK and build tools
.......................................

For preparing the SDK and building the cctools, we use the very helpful
scripts from the `OS X Cross <https://github.com/tpoechtrager/osxcross>`
toolchain. If you re interested in the details, and what other features OS X
Cross offers, please refer to it's homepage.

Side-note: For actually accessing the OS X disk image file (`.dmg`),
`darling-dmg https://github.com/darlinghq/darling-dmg` is used. It allows
mounting `.dmg`s under Linux via FUSE.

For saving you reading OSXCross' documentation we prepared a virtual box
description performing all required steps.
If you are interested in the precise commands, please refer to
`packages_osxcross_debianoid`, `prepare_osxcross_debianiod`, and
`build_osxcross` FIXME the Vagrantfile.

Please proceed as follows:

1. Download `XCode 7.3.x
   <https://developer.apple.com/downloads/index.action?name=Xcode%207.3` and
   save it to :file:`bootloader/sdks/osx/`. You will need to register an
   `Apple ID`, for which you may use a disposable e-mail-address, to search
   and download the files.

   Please make sure that you are complying to the license of the respective
   package.

2. Use the Vagrantfile to automatically build the SDK and tools::

     vagrant up build-osxcross && vagrant halt build-osxcross

   This should create the file :file:`bootloader/sdks/osx/osxcross.tar.xz`,
   which will then be installed on the build-host.

   If for some reason this fails, try running `vagrant provision
   build-osxcross`.

3. This virtual machine is no longer used, you may now want to discard it
   using ``vagrant destroy build-osxcross``.


Building the Bootloader
.......................................

Again, simply use the Vagrantfile to automatically build the OS X bootloaders::

     vagrant up linux64 && vagrant halt linux

This should create the bootloaders in
* :file:`../PyInstaller/bootloader/Darwin-{*}/`.

   If for some reason this fails, try running `vagrant provision
   linux64`.

3. This virtual machine is no longer used, you may now want to discard it
   using ``vagrant destroy build-osxcross``.


On the build host perform the following steps::

    cd /vagrant/bootloader
    mkdir -p ~/osxcross
    tar -C ~/osxcross --xz -xf /vagrant/sdks/osx/osxcross.tar.xz
    PATH=~/osxcross/bin/:$PATH
    python ./waf all CC=o64-clang
    python ./waf all CC=o32-clang


PATH=~/osxcross/target/bin/:$PATH
python ./waf CC=x86_64-apple-darwin15-clang all


Verifying results
.......................................


alias otool=~/osxcross/bin/x86_64-apple-darwin15-otool
cd /vagrant/PyInstaller/bootloader/
for i in Darwin-*bit/{run,runw,run_d,runw_d} ; do
	otool -aSfhlLDtdOrITRMHGCPvV $i > $i.txt
	# -q -Q
done



Building for Windows
~~~~~~~~~~~~~~~~~~~~~~~~

The pre-compiled |bootloader| coming with PyInstaller is a
self-contained static executable that imposes no restrictions
on the version of Python being used.

When building the bootloader yourself, you have carefully choose
between two options:

1. Using the Visual Studio C++ compiler.

   This allows creating self-contained static executables,
   which can be used for all versions of Python.
   This is why the bootloaders delivered with PyInstaller are build using
   Visual Studio C++ compiler.
 
   You can use any Visual Studio version that is convenient
   (as long as it's supported be the waf build tool).


2. Using the `MinGW-w64`_ suite.

   This allows to create smaller, dynamically linked executables,
   but requires to use the same
   level of Visual Studio [*]_
   as was used to compile Python.
   So this bootloader will be tied to a specific version of Python.

   The reason for this is, that unlike unix-like systems, Windows doesn’t
   supply a system standard C library, 
   leaving this to the compiler. 
   But Mingw-w64 doesn’t have a standard C library. 
   Instead it links against msvcrt.dll, which happens to exist
   on many Windows installations – but i not guaranteed to exist.

.. [*] This description seems to be technically incorrect. I ought to depend
       on the C++ run-time library. If you know details, please open an
       |issue|.


Build using Visual Studio C++
---------------------------------

* With our `wscript` file, you don't need to run ``vcvarsall.bat`` to ’switch’
  the environment between VC++ installations and target architecture. The
  actual version of C++ does not matter and the target architecture is
  selected by using the ``--target-arch=`` option.

* If you are not using Visual Studio for other work, installing only the
  standalone C++ build tools might be the best option as it avoids bloading
  your system with stuff you don't need (and saves *a lot* if installation
  time).

  .. note:
     Hint: We recommend
     installing the build tools software using the
     `chocolate <...>`_ package manager.
     While at a first glance it looks like overdose, this is the easiest
     way to to install the C++ build-tools. It comes town to two lines in an
     administrative powershell::

       … one-line install as written on the chocolate homepage
       choco install -y ... FIXME
       choco install -y python  # just in case you don't have it :-)

* Useful Links:

  * `Microsoft Visual C++ Build Tools 2015
    <http://landinghub.visualstudio.com/visual-cpp-build-tools>`_
  * `Microsoft Build Tools for Visual Studio 2017.
    <https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2017>`_



Build using MinGW-64
-----------------------

Please be aware of the restrictions mentioned above.

If Visual Studio is not convenient,
you can download and install the MinGW distribution from one of the
following locations:

* `MinGW-w64`_ required, uses gcc 4.4 and up.

* `TDM-GCC`_ - MinGW (not used) and MinGW-w64 installers

On Windows, when using MinGW-w64, add ``PATH_TO_MINGW\bin``
to your system ``PATH``. variable. Before building the
|bootloader| run for example::

        set PATH=C:\MinGW\bin;%PATH%


If you have installed both Visual C++ and MinGW, you might need to add run
``waf --gcc …``.



.. include:: _common_definitions.txt

.. Emacs config:
 Local Variables:
 mode: rst
 ispell-local-dictionary: "american"
 End:
