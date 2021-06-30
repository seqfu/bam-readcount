Build instructions
==================


Requirements
------------

    A C++ toolchain
    cmake 
    make

On a Debian Linux-based system such as Ubuntu 

    apt install build-essential cmake

will install the required software, or see `Build (Docker)` below.

Builds are currently failing under OS X: see `OS X` below.

All required libraries are included in the repository under `vendor/`.
See [vendor/README.md](vendor/README.md) for more information.


Build
-----

Make a build directory

    mkdir build

Run CMake from inside it

    cd build
    cmake ..

Run Make

    make 

This will build all the vendored libraries as well as `bam-readcount`.
The final binary, which can be moved anywhere, is

    bin/bam-readcount

Try it on a test CRAM

    cd ../test-data
    ../build/bin/bam-readcount -f rand1k.fa twolib.sorted.cram


Build (Docker)
--------------

This is mainly useful when working on the `bam-readcount` code.
  
If you have Docker running, build using the included Docker image

    cd docker/minimal-cmake
    make interact

This will start a minimal Docker container with `build-essential` and
`cmake` installed, with this cloned repository mounted at 

    /bam-readcount

and starting in that directory. Then follow the build instructions 
above.




Docker container
----------------

For development builds, use the method above. This method will remove
and rebuild from scratch every time.

To run the latest build from DockerHub on the test data

    cd test-data

    # Will mount current directory as /work and start in that directory
    # in order to find the input files
    docker run -v $(pwd):/work -w /work seqfu/bam-readcount /bin/bam-readcount -f rand1k.fa twolib.sorted.cram

This is a two-stage build, copying the `bam-readcount` binary from the
first stage (with build tools) into a minimal image as
`/bin/bam-readcount`. To run the build yourself

    cd docker/bam-readcount
    make build
    # Try out interactively
    make interact
    # Change the NAME at the top of the Makefile first
    # push to DockerHub with git hash as tag and also set to latest
    make push


Test data
---------

There is a small two-library test CRAM file

    test-data/twolib.sorted.cram  

with associated reference

    test-data/rand1k.fa

The reference is encoded in the CRAM as 

    @SQ	SN:rand1k	LN:1000	M5:11e5d1f36a8e123feb3dd934cc05569a	UR:rand1k.fa

so `bam-readcount` should be run inside the `test-data` directory to
find the reference.


OS X
----

OS X builds fail for (at least) two reasons on our High Sierra machine
and under GitHub Actions with `mac-10.15` (Catalina) and `mac-11` (Big
Sur).

When linking Boost, there are pages of errors.

This is also true of the current `genome/bam-readcount` `master`.


Todo
----

Add `URL_HASH` for vendored libraries for CMake verification.

On MGI's LSF cluster, the curl/https lookup in the ENA CRAM registry
fails. This might be expected due to network restrictions on cluster
containers.

