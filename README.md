# Additional Files

# Cookbook

Run all iVar commands using two Zika virus replicates with known reference sequences - 26_a and 26_b.

Download required files [here](https://www.dropbox.com/sh/a350izwgxkgxcxj/AAC8JtI-jmEF4xLV3M1C2Qwxa?dl=0)

## INSTALLATION

Xcode command line tools install
```
xcode-select --install
[follow prompts]
```

### Brew install
```
https://brew.sh/
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Autotools install
```
brew install autoconf automake libtool
```

### Samtools install

Download SAMtools from [here](http://www.htslib.org/download/) or [Github](https://github.com/samtools/samtools)

```
autoheader
autoconf -Wno-syntax
./configure
make
make install
```

### htslib install

Download htslib from [here](http://www.htslib.org/download/) or [Github](https://github.com/samtools/htslib)

```
autoheader
autoconf
./configure
make
make install
```

### iVar install

Download iVar from [Github](https://github.com/andersen-lab/ivar)

```
./autogen.sh
./configure
make
make install
```

### Running iVar

```
# Index bam files
samtools index ZI-merge-26_a.sorted.bam
samtools index ZI-merge-26_b.sorted.bam

# Trim primers using iVar
ivar trim -i ZI-merge-26_a.sorted.bam -b ZKV_primers.bed -p 26_a.trimmed
ivar trim -i ZI-merge-26_b.sorted.bam -b ZKV_primers.bed -p 26_b.trimmed

# Sort output bam files
samtools sort 26_a.trimmed.bam > 26_a.trimmed.sorted.bam
samtools sort 26_b.trimmed.bam > 26_b.trimmed.sorted.bam

# Index output bam files
samtools index 26_a.trimmed.sorted.bam
samtools index 26_b.trimmed.sorted.bam

# Merge output bam files for consensus calling
samtools merge 26.trimmed.merged.bam 26_a.trimmed.sorted.bam 26_b.trimmed.sorted.bam

# Call consensus sequence from the merged replicates using iVar
samtools mpileup -A -d 300000 -Q 0 -F 0 26.trimmed.merged.bam | ivar consensus -p 26.consensus

# Call iSNVs with iVar from the two replicates
samtools mpileup -A -d 300000 --reference ZIKV_PRV.fasta -Q 0 -F 0 26_a.trimmed.sorted.bam | ivar variants -t 0.05 -p 26_a.variants
samtools mpileup -A -d 300000 --reference ZIKV_PRV.fasta -Q 0 -F 0 26_b.trimmed.sorted.bam | ivar variants -t 0.05 -p 26_b.variants

# Mask amplicons from primers with mismatches
ivar getmasked -i 26_a.variants.tsv -b ZKV_primers.bed -p 26_a.getmasked
ivar getmasked -i 26_b.variants.tsv -b ZKV_primers.bed -p 26_b.getmasked

# Remove amplicons from primers with mismatches
ivar removereads -i 26_a.trimmed.sorted.bam -p 26_a.trimmed.sorted.filtered -t 26_a.getmasked.txt
ivar removereads -i 26_b.trimmed.sorted.bam -p 26_b.trimmed.sorted.filtered -t 26_b.getmasked.txt

# Call iSNVs with iVar from the two replicates with mismatched amplicons removed
samtools mpileup -A -d 300000 --reference ZIKV_PRV.fasta -Q 0 -F 0 26_a.trimmed.sorted.filtered.bam | ivar variants -t 0.05 -p 26_a.variants.filtered
samtools mpileup -A -d 300000 --reference ZIKV_PRV.fasta -Q 0 -F 0 26_b.trimmed.sorted.filtered.bam | ivar variants -t 0.05 -p 26_b.variants.filtered

# Filter variants from the two replicates
ivar filtervariants -p 26.filtered_variants 26_a.variants.filtered.tsv 26_b.variants.filtered.tsv
```
