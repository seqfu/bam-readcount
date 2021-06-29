bam-readcount
=============

![tests](https://github.com/seqfu/bam-readcount/actions/workflows/tests.yml/badge.svg?branch=actions-config)
![coverage](https://coveralls.io/repos/seqfu/bam-readcount/badge.svg?branch=actions-config&service=github)


`bam-readcount` runs on a `BAM` or `CRAM` file and generates metrics at
single nucleotide positions. These metrics can be useful for filtering
out false positive variant calls.

For build instructions, see [BUILD.md](BUILD.md).

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
position:

    chr	position	reference_base	depth	base:count:avg_mapping_quality:avg_basequality:avg_se_mapping_quality:num_plus_strand:num_minus_strand:avg_pos_as_fraction:avg_num_mismatches_as_fraction:avg_sum_mismatch_qualities:num_q2_containing_reads:avg_distance_to_q2_start_in_q2_reads:avg_clipped_length:avg_distance_to_effective_3p_end   ...

There is one set of `:`-separated fields for each reported `base`:

Field | Description
----- | -----------
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
