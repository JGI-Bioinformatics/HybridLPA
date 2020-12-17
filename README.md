# HybridLPA

**SpaRC** (refer to [paper](https://academic.oup.com/bioinformatics/article/35/5/760/5078476) and [sources](https://bitbucket.org/LizhenShi/sparc/src/master/)) is capable of forming accurate clusters in terms of both purity and completeness on the long reads. However, it suffers from the under clustering issue on the illumina short reads. In order to solve this issue, we leverage the long connections of the long reads to help clustering the short reads. We optimized original LPA for clustering metagenomic long and short reads. In the HybridLPA, we only allow the long reads labels to propogate throughout the read graph. This site provides MPI/UPC++ based implementation of **HybridLPA**.




## Installation



First clone the code

```
    git clone https://github.com/JGI-Bioinformatics/HybridLPA.git
    cd HybridLPA && git submodule update --init --recursive
```
The two versions are independent from each other, you may choose to only build the version that you are interested in.


### MPI Version

#### Requires
* Linux 
* mpi >= 3.0
* c++ standard >= 11 


#### Build
```
    mkdir build && cd build
    cmake -DBUILD_SPARC_MPI=ON ..
    make sparc_mpi
```
### UPC++ Version

#### Requires
* Linux 
* mpi >= 3.0
* c++ standard >= 11 
* upcxx 2020.3.2


#### Build

Install UPC++ is beyond this scope. 
Set environment *UPCXX_INSTALL* helps cmake to find upcxx.
Also remember set UPCXX_NETWORK to your network, otherwise smp is most likely be used. 
```
    mkdir build && cd build
    cmake -DBUILD_SPARC_UPCXX=ON ..
    make sparc_upcxx
```


## Usages

Similar to [Sparc](https://bitbucket.org/LizhenShi/sparc/src/master/README.md),  given a sequence file the flow of analysis includes 4 steps. For each program use "-h" or "--help" option to get help.

### Kmer Counting (optional)

Find the kmer counting profile of the data, so that we decides how to filter out "bad" kmers. In this step we run edge_generating_$SURFIX where $SURFIX means mrmpi, mimir, mpi or upcxx. 

For example for mpi version:
```
    $./kmer_counting_mpi -h
    
    -h, --help
    shows this help message
    -i, --input
    input folder which contains read sequences
    -p, --port
    port number
    -z, --zip
    zip output files
    -k, --kmer-length
    length of kmer
    -o, --output
    output folder
    --without-canonical-kmer
    do not use canonical kmer

```


### Kmer-Reads-Mapping

Find shared reads for kmers with kmer_read_mapping_$SURFIX where $SURFIX means mrmpi, mimir, mpi or upcxx. 

For example for mpi version:
```
    $./kmer_read_mapping_mpi -h
    
    -h, --help
    shows this help message
    -i, --input
    input folder which contains read sequences
    -p, --port
    port number
    -z, --zip
    zip output files
    -k, --kmer-length
    length of kmer
    -o, --output
    output folder
    --without-canonical-kmer
    do not use canonical kmer

```

### Edge Generating

Generate graph edges using edge_generating_$SURFIX where $SURFIX means mrmpi, mimir, mpi or upcxx. 

For example for mpi version:
```
    $./edge_generating_mpi -h
    
    -h, --help
    shows this help message
    -i, --input
    input folder which contains read sequences
    -p, --port
    port number
    -z, --zip
    zip output files
    -o, --output
    output folder
    --max-degree
    max_degree of a node; max_degree should be greater than 1
    --min-shared-kmers
    minimum number of kmers that two reads share. (note: this option does not work)
    
```

### LPA Clustering

Find clusters by Label Propagation Algorithm. 
In this step there are two options: PowerGraph LPA or *lpav1_mpi/upcxx*. 

*lpav1_mpi/upcxx* is my implementation of LPA in MPI or UPC++, which is build in the corresponding targets.

PowerGraph(https://github.com/jegonzal/PowerGraph) also provide LPA program which can be got from github.

*Remark:* Since this step is purely graph clustering, actually any graph clustering algorithms can be used providing it can work on graphs of billions edges.

## Example Runs

Please find sbatch scripts of sample runs on [LAWRENCIUM](https://sites.google.com/a/lbl.gov/high-performance-computing-services-group/lbnl-supercluster/lawrencium) in [misc/example](misc/example) folder.

When running the programs, make sure there are enough nodes to hold all the data in memory. Although some programs support storing temporary data in disk, but it will make the progress quite slow.

