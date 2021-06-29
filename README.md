bam-readcount
=============

![tests](https://github.com/seqfu/bam-readcount/actions/workflows/tests.yml/badge.svg?branch=actions-config)
![coverage](https://coveralls.io/repos/seqfu/bam-readcount/badge.svg?branch=actions-config&service=github)


`bam-readcount` runs on a `BAM` or `CRAM` file and generates metrics at
single nucleotide positions. These metrics can be useful for filtering
out false positive variant calls.

Run with no arguments for command-line help:

    $ bam-readcount

    Usage: bam-readcount [OPTIONS] <bam_file> [region]
    Generate metrics for bam_file at single nucleotide positions.
    Example: bam-readcount -f ref.fa some.bam

    Available options:
      -h [ --help ]                         produce this message
      -v [ --version ]                      output the version number
      -q [ --min-mapping-quality ] arg (=0) minimum mapping quality of reads used
                                            for counting.
      -b [ --min-base-quality ] arg (=0)    minimum base quality at a position to
                                            use the read for counting.
      -d [ --max-count ] arg (=10000000)    max depth to avoid excessive memory
                                            usage.
      -l [ --site-list ] arg                file containing a list of regions to
                                            report readcounts within.
      -f [ --reference-fasta ] arg          reference sequence in the fasta format.
      -D [ --print-individual-mapq ] arg    report the mapping qualities as a comma
                                            separated list.
      -p [ --per-library ]                  report results by library.
      -w [ --max-warnings ] arg             maximum number of warnings of each type
                                            to emit. -1 gives an unlimited number.
      -i [ --insertion-centric ]            generate indel centric readcounts.
                                            Reads containing insertions will not be
                                            included in per-base counts


Output
------

Output is tab-separated with no header to `STDOUT`, one line per
position as follows:

    chr	position	reference_base	depth	base:count:avg_mapping_quality:avg_basequality:avg_se_mapping_quality:num_plus_strand:num_minus_strand:avg_pos_as_fraction:avg_num_mismatches_as_fraction:avg_sum_mismatch_qualities:num_q2_containing_reads:avg_distance_to_q2_start_in_q2_reads:avg_clipped_length:avg_distance_to_effective_3p_end   ...

    |
--- | ---
base | the base that all reads following in this field contain at the reported position i.e. C
count | the number of reads containing the base
avg_mapping_quality | the mean mapping quality of reads containing the base
avg_basequality | the mean base quality for these reads
avg_se_mapping_quality | mean single ended mapping quality
num_plus_strand | number of reads on the plus/forward strand
num_minus_strand | number of reads on the minus/reverse strand
avg_pos_as_fraction | average position on the read as a fraction (calculated with respect to the length after clipping). This value is normalized to the center of the read (bases occurring strictly at the center of the read have a value of 1, those occurring strictly at the ends should approach a value of 0)
avg_num_mismatches_as_fraction | average number of mismatches on these reads per base
avg_sum_mismatch_qualities | average sum of the base qualities of mismatches in the reads
num_q2_containing_reads | number of reads with q2 runs at the 3’ end
avg_distance_to_q2_start_in_q2_reads | average distance of position (as fraction of unclipped read length) to the start of the q2 run
avg_clipped_length | average clipped read length of reads
avg_distance_to_effective_3p_end | average distance to the 3’ prime end of the read (as fraction of unclipped read length)


This branch
-----------

This branch is a refactor of `bam-readcount` to the `samtools-1.10` 
(and `htslib-1.10`) API, which adds CRAM support. Should build in a 
minimal environment with C and C++ compilers, Make, and CMake.

To clone this fork and branch 

    git clone -b samtools-1.10 https://github.com/seqfu/bam-readcount
    cd bam-readcount

Builds are failing under OS X, see `OS X` below.


Vendored libraries
------------------

Any build requires downloading the vendored libraries.

Already under `vendor/` are
  
    vendor/
      boost-1.55-bamrc.tar.gz         # subset of Boost needed for bamrc
      Makefile.disable_curl.patch     # Not used

The patch is no longer used.  Except for Boost the vendored libraries
are not included in the repository. To fetch them, run 

    # wget is required for this script
    0/populate_vendor.sh

This should download

    vendor/
      bzip2-1.0.8.tar.gz
      curl-7.67.0.tar.gz
      mbedtls-2.16.4-apache.tgz
      samtools-1.10.tar.bz2
      xz-5.2.4.tar.gz
      zlib-1.2.11.tar.gz

I chose `mbedtls-2.16.4-apache.tgz` for `libcurl` SSL support as it has
no additional dependencies. This is used (via `https`) to query the ENA
CRAM registry for reference hashes. Another option might be wolfSSL's
tiny-cURL distribution, which builds a lighter cURL with HTTPS support,
but the download process involves a form.


Build
-----

Before building, make sure to download the vendored libraries as in the
section above.


### Run minimal build Docker container

If you have Docker running, build using the included Docker image

    cd docker/minimal-cmake
    # This will start a container using image seqfu/minimal-cmake
    make interact

This will start a minimal docker container with `build-essential` and
`cmake` installed, with the cloned repository mounted at 

    /bam-readcount

and starting in that directory. 


### Build `bam-readcount` and vendored libraries

Make a build directory

    mkdir build

Run CMake from inside it

    cd build
    cmake ..

Run Make

    make 

This will build all the vendored libraries as well as `bam-readcount`.
The final binary will be

    bin/bam-readcount

Try it on a test CRAM

    cd ../test-data
    ../build/bin/bam-readcount -f rand1k.fa twolib.sorted.cram


### Test data

There is a small two-library test CRAM file

    test-data/twolib.sorted.cram  

with associated reference

    test-data/rand1k.fa

The reference is encoded in the CRAM as 

    @SQ	SN:rand1k	LN:1000	M5:11e5d1f36a8e123feb3dd934cc05569a	UR:rand1k.fa

so `bam-readcount` should be run inside the `test-data` directory to
find the reference.


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


CRAM reference
--------------

If no reference is given to `bam-readcount` with `-f`, the CRAM will
attempt to use the reference encoded in the header, or do a lookup at
ENA (this seems to have some issues when run on the LSF cluster). 

If a reference is given with `-f FASTA`, it will override whatever is 
in the CRAM header.


Todo
----

Tested against `genome/bam-readcount` `master` with a simple BAM file 
converted to CRAM with identical output, but needs more testing.

`find_library_names()` line 91 in 

    src/exe/bam-readcount/bamreadcount.cpp

has been modified to return an empty list because `sam_header2key_val`
has been removed (along with all of `sam_header.h`) and there is a new
(`hrecs`?) API to access header data that we will need to use. Enabling
`-p` still appears to work. It looks like the list is only used to print
expected library names to `STDERR`.

Add `URL_HASH` for vendored libraries for CMake verification.

Lower CMake minimum version requirement in `BuildSamtools.cmake`, 
and be consistent throughout.

On MGI's LSF cluster, the curl/https lookup in the ENA CRAM registry
fails. This might be expected due to network restrictions on cluster
container.


OS X
----

OS X builds fail for (at least) two reasons on my High Sierra machine.
This may be due to my setup with MacPorts etc; I haven't looked into
this much and the observations below might not be relevant.

Boost: 

When linking, I get pages of errors.

This is also true of the current `genome/bam-readcount` `master`.

cURL: 

Not sure what goes wrong here, Make just exits with

    make: *** [all] Error 2

I disabled `libcurl` for a while using the Makefile patch under

    vendor/Makefile.disable_curl.patch

Also getting errors 

    bam-readcount/src/lib/bamrc/BasicStat.hpp:5:10: fatal error:
          'sam.h' file not found
    #include "sam.h"
             ^~~~~~~
    1 error generated.

Not sure why, since it seems to be a build configuration problem but
doesn't happen in the minimal Docker build container.




