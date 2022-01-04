Build
=====

These scripts perform two functions
# create build for the intended platform 
# transfer build output to cert.mcs.cmu.edu:/usr/share/asceds/<platform>

Notes:
# These scripts use the source code in the ~/git/asceds ~/build directory (hardcoded) 

.. _build:

* asceds-build-arch
* asceds-build-arch-dev
* asceds-build-deb
* asceds-build-rpm

    Description: This script can be used to build and release packages for centOS.
    How:
    
    # Connects to a centOS host centos7.math.cmu.edu (hardcoded)
    # Uses rsync to sync below dirs
        - ./dist
        - ./pack/REDHAT
        - ./asceds/asceds-build-rpm-dev
    # Calls the asceds-build-rpm-dev to perform the actual build
    # Transfers the build output to cert.mcs.cmu.edu

* asceds-build-rpm-dev

    Description: This script can be used on centOS machine to build RPM package
    How:
    
    # Picks up the version from dist/usr/lib/asceds/etc/version and checks version format
    # Updates version number in pack/ARCH/PKGBUILD
    # Removes build/asceds

    Dependencies:
    yum install makepkg 
