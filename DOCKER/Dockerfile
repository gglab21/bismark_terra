FROM ubuntu:21.10

ARG VERSION=__specify_in_build_arg__

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt upgrade -y
 
RUN apt-get install -y procps build-essential zlib1g-dev python3 python3-dev\
                       liblzma-dev libbz2-dev libxml2-dev libncurses5-dev libncursesw5-dev git wget unzip \
                       python3-pip python-is-python3 \
                       libssl-dev libcurl4-openssl-dev dirmngr gnupg \
                       apt-transport-https ca-certificates software-properties-common\
                       fastqc pandoc 

# Install Bowtie2
RUN apt-get install -y libtbb-dev && \
	mkdir -p /src && \
	cd /src && \
	wget -O bowtie2.zip https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.4.5/bowtie2-2.4.5-linux-x86_64.zip/download && \
	unzip bowtie2.zip && \
	ln -s /src/bowtie2-2.4.5-linux-x86_64/bowtie2* /usr/local/bin

# Install R
#RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
#	add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/' && \
#	apt-get update && \
#	apt-get install -y --force-yes r-base
RUN apt-get update && apt-get install -y --no-install-recommends software-properties-common dirmngr && \
    wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc && \
    add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/" && \
    apt-get install -y --no-install-recommends r-base && \
    add-apt-repository ppa:c2d4u.team/c2d4u4.0+

RUN Rscript -e "install.packages('BiocManager');" && \
    Rscript -e "install.packages('optparse', repos='http://R-Forge.R-project.org')" && \
    Rscript -e "install.packages('ggplot2',repos='http://R-Forge.R-project.org')" && \
    Rscript -e "install.packages('foreach',repos='http://R-Forge.R-project.org')" && \
    Rscript -e "install.packages('yaml',repos='https://mirror.las.iastate.edu/CRAN/')" && \
    Rscript -e "install.packages('devtools',repos='https://mirror.las.iastate.edu/CRAN/')" && \
    Rscript -e "install.packages('rmarkdown',repos='https://mirror.las.iastate.edu/CRAN/')" && \
    Rscript -e "BiocManager::install('bsseq');" && \
    Rscript -e "BiocManager::install('BiocGenerics');" && \
    Rscript -e "BiocManager::install('XML')" && \
    Rscript -e "BiocManager::install('genefilter')" && \
    Rscript -e "BiocManager::install('rtracklayer')" && \
    Rscript -e "BiocManager::install('BSgenome')" && \
    Rscript -e "BiocManager::install('Ringo')" && \
    Rscript -e "BiocManager::install('Repitools')" && \
    Rscript -e "BiocManager::install('knitr')" && \
    Rscript -e "BiocManager::install('AnnotationHub')" && \
    Rscript -e "BiocManager::install('BSgenome.Mmusculus.UCSC.mm10')" && \
    Rscript -e "BiocManager::install('annotatr')" && \
    Rscript -e "BiocManager::install('scmeth')" 

# Install aggregation workflow R packages
RUN Rscript -e "BiocManager::install(c(\
            'BSgenome', 'Biostrings','GenomeInfoDb', \
            'HDF5Array', 'bsseq', \
            'foreach', 'optparse', 'annotatr', \
            'readr', 'stringr', 'devtools', \
            'TxDb.Mmusculus.UCSC.mm9.knownGene', \
            'TxDb.Mmusculus.UCSC.mm10.knownGene', \
            'org.Mm.eg.db', \
            'TxDb.Hsapiens.UCSC.hg19.knownGene', \
            'TxDb.Hsapiens.UCSC.hg38.knownGene', \
            'org.Hs.eg.db', \
            'scmeth'), ask=F)"

# Install bedtools
RUN cd /src && \
    wget https://github.com/arq5x/bedtools2/releases/download/v2.30.0/bedtools-2.30.0.tar.gz && \
    tar -zxf bedtools-2.30.0.tar.gz && \
    cd bedtools2 && \
    make && \
	ln -s /src/bedtools2/bin/bedtools /usr/local/bin

# Install samtools
RUN cd /src && \
    git clone https://github.com/samtools/htslib.git && \
    cd htslib && git submodule update --init --recursive &&\
    git checkout tags/1.14 && \
    make && \
    cd .. && \
    git clone https://github.com/samtools/samtools.git && \
    cd samtools && \
    git checkout tags/1.14 && \
    make && \
    ln -s /src/samtools/samtools /usr/local/bin

# Install bedgraphToBigwig
RUN cd /usr/local/bin && \
    wget http://hgdownload.soe.ucsc.edu/admin/exe/linux.x86_64/bedGraphToBigWig && \
    chmod +x bedGraphToBigWig

# Install cutadapt (for Trimgalore)
RUN pip3 install cutadapt

# Install Trimgalore
RUN cd /tmp && \
    wget https://github.com/FelixKrueger/TrimGalore/archive/0.6.7.zip && \
    unzip 0.6.7.zip && \
    cp TrimGalore-0.6.7/trim_galore /usr/local/bin && \
    chmod +x /usr/local/bin/trim_galore

# Install Bismarkhttps://github.com/FelixKrueger/Bismark/archive/
RUN cd /src && \
	wget https://github.com/FelixKrueger/Bismark/archive/0.23.1.tar.gz && \
	tar zxf 0.23.1.tar.gz && \
	ln -s /src/Bismark-0.23.1/bismark* /usr/local/bin/ && \
	ln -s /src/Bismark-0.23.1/deduplicate_bismark /usr/local/bin/            
            
# Add pooling report script
ADD ./pool_report_files.R pool_report_files.R

# Add combine individual samples script
ADD ./bismark_to_bsseq_file_list.R bismark_to_bsseq_file_list.R


RUN echo $VERSION > /VERSION
