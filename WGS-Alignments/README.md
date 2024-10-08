# WGSpipelines

## Description
This pipeline is designed for whole genome sequencing (WGS) data alignment and variant calling. It utilizes Sentieon tools for high-performance alignment, duplicate removal, and base quality score recalibration (BQSR), and supports multiple downstream variant calling workflows, including Strelka2 and Sentieon GATK4.

## Requirements
- Sentieon
- Strelka2
- Python
- ATLAS (in-house data management software)
- SLURM Workload Manager (for job memory and CPU allocation)

## Usage
Run the wdl with cromwell using cromwell submit with the input wdl and a json containing the required inputs.

```bash
cromwell submit wgs_alignment_pipeline.wdl <input json>
```

## Arguments
- halb: HALB id for sample.
- extra_halbs: Additional HALB ids to process.
- extra_fastqs: Additional FASTQ files to include in the analysis.
- use_atlas: Boolean to enable or disable the ATLAS in-house data management system.
- reference_fa and associated files: Reference genome files.
- license_server: Sentieon license server.
- write_recal, run_strelka, run_gatk, write_cram: Booleans to control the optional steps (e.g., writing recalibrated BAM files, running Strelka2 or GATK variant calling).

## Output
The pipeline generates several output files based on the steps executed:

- BAM/CRAM Files: Aligned and optionally recalibrated sequence data.
- VCF Files: VCFs generated by Strelka2 or Sentieon GATK4.
- Quality Metrics: Output files containing alignment statistics.

## Script Components
- LocateFastqs: Task to locate and optionally download FASTQ files using ATLAS.
- SentieonPostalt: Alignment task using Sentieon.
- SentieonDedup: Task for deduplicating.
- SentieonRecal: Base quality score recalibration (BQSR) task.
- Strelka2: Variant calling using Strelka2.
- SentieonGATK4: Variant calling using Sentieon GATK4.
- CompileFastp: Task to compile FASTP output.
- BamToCram: Task to generate a CRAM.

## Additional Notes
- Make sure the paths to the reference genome files and other required inputs are correctly specified in your environment.
- Update any SLURM job attributes to fit the needs of your analysis.

## References
- Freed, D., Aldana, R., Weber, J. A., & Edwards, J. S. (2017). The Sentieon Genomics Tools - A fast and accurate solution to variant calling from next-generation sequence data. bioRxiv, 115717. https://doi.org/10.1101/115717 
- Kim, S., Scheffler, K., Halpern, A.L. et al. Strelka2: fast and accurate calling of germline and somatic variants. Nat Methods 15, 591–594 (2018). https://doi.org/10.1038/s41592-018-0051-x 

## Contact
For support or contributions, please contact me at:
 
`jtaylor[at]hudsonalpha.org`
