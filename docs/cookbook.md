# Cookbook

## Custom reference database
### From a DFAST reference file  
1. Create a reference file  
Prepare a TAB-separated reference file following the format described in the [Reference Database](workflow.md) section.  
The file extension must be '.ref', say 'your_reference.ref'.
2. Format database  
    ```
    python $DFAST_APP_ROOT/scripts/reference_util.py formatdb -i your_reference.ref
    ```
    Then, index files for GHOSTX and BLASTP will be generated in the same location as the reference file.
3. Configure and run  
    Specify the reference file in the `database` attribute in the 'DBsearch' section,  
    e.g. `"database": "/path/to/your_reference.ref"`

    Alternatively, you can run dfast using the `--database` option.
    ```
    dfast --genome your_genome.fa --database /path/to/your_reference.ref
    ```

### From a FASTA file  
You can prepare a database easily from a FASTA file using 'reference_util.py'.  
The script can parse FASTA definition lines for NCBI/UniprotKB/Prokka styles.
 
1. Convert a FASTA file into DFAST reference format
    ```
    python $DFAST_APP_ROOT/scripts/reference_util.py fasta2dfast -i your_reference.fasta -o your_reference.ref
    ```
2. Format database, configure, and run  
Then, follow the same procedure as above.

## OrthoSearch
OrthoSearch identifies orthologous genes based on a simple Reciprocal-Best-Hit (RBH) approach.
This is effective in reducing running time and in transferring annotations from a reference genome of the closely-related organism.  
This recipe shows how to perform OrthoSearch.

### Reference proteome
OrthoSearch requires a 'reference proteome' file that contains all protein sequences in a genome.
The file format must be either of FASTA, GenBank, or DFAST reference format. 
In addition to a plain FASTA format (sequence ID and definition), OrthoSearch can parse FASTA definition lines of UniProt, GenBank, and Prokka styles.  
The format is automatically recognized.

### Recipe
Our recommendation is to download a GenBank-format file from the NCBI Assembly Database and to use it as a reference. 
1. Download a reference proteome  
    ```
    python $DFAST_APP_ROOT/scripts/file_downloader.py --assembly GCF_000005845 
    ```
    This will download the latest version of the Escherichia coli str. K-12 genome in a GenBank-format into the current directory with the file name 'GCF_000005845.2.gbk'.
    You can use the `--out` option to specify the directory into which the file is downloaded.
2. Run DFAST  
    Use `--references` to specify the reference proteome(s).
    ```
    dfast --genome your_genome.fa --references GCF_000005845.2.gbk
    ```
    You can specify multiple proteome files with commas to separate files. 
    ```
    dfast --genome your_genome.fa --references GCF_000005845.2.gbk,GCA_000008865.1.faa
    ```
    When multiple files are used as references, 
    all-vs-all alignments are conducted between a query proteome and each of the reference proteomes, 
    and the highest-scoring hit will be adopted as the result.
* Configuration  
Reference proteomes can also be specified in the configuration file.  
Set `enabled` to True, and specify `references` in the 'FUNCTIONAL_ANNOTATION' part.
    ```
    {
        "component_name": "OrthoSearch",
        "enabled": True,
        "options": {
            # "cpu": 2,  # Uncomment this to set the component-specific number of CPUs.
            "skipAnnotatedFeatures": False,
            "evalue_cutoff": 1e-6,
            "qcov_cutoff": 75,
            "scov_cutoff": 75,
            "aligner": "ghostx",
            "aligner_options": {},
            "references": ["GCF_000005845.2.gbk", "GCA_000008865.1.faa"]
        },
    },
    ```
## BlastSearch
BlastSearch is for protein homology search against a large-sized reference database, such as pre-formatted Blast databases like RefSeq Protein and SwissProt available at the NCBI FTP site.
### How to use a pre-formatted BLAST database
1. Download a database from NCBI  
    ```
    wget ftp://ftp.ncbi.nlm.nih.gov//blast/db/swissprot.tar.gz
    tar xvfz swissprot.tar.gz
    ```
2. Create a configuration file  
Set `enabled` to True, and specify `database` to be searched against.
You can also specify `dbtype`, but normally, leaving it 'auto' will do.  
Place this part upstream of 'DBsearch' against the default database
 if you want to give priority to 'BlastSearch'.
    ```
    {
        "component_name": "BlastSearch",
        "enabled": True,
        "options": {
            # "cpu": 2,  # Uncomment this to set the component-specific number of CPUs.
            "skipAnnotatedFeatures": False,
            "evalue_cutoff": 1e-6,
            "qcov_cutoff": 75,
            "scov_cutoff": 75,
            "aligner": "blastp",  # Must be blastp
            "aligner_options": {},
            "dbtype": "auto",  # Must be either of auto/ncbi/uniprot/plain
            "database": "/path/to/swissprot",
        },
    },
    ```
3. Run DFAST
    ```
    dfast --genome your_genome.fa --config your_config.py
    ```
### Prepare database from a FASTA file
Here is an example to create a database for RefSeq nonredundant archaeal proteins.
1. Download FASTA files  
    ```
    wget ftp://ftp.ncbi.nlm.nih.gov//refseq/release/archaea/archaea.nonredundant_protein.*.protein.faa.gz
    gunzip -c archaea.nonredundant_protein.*.protein.faa.gz > archaea.nonredundant_protein.faa
    ```
2. Format database  
Be sure to use `-parse_seqids`.
    ```
    makeblastdb -hash_index -parse_seqids -dbtype prot -in archaea.nonredundant_protein.faa
    ```
3. Create a configuration file and run DFAST  
Follow the recipe described above.

