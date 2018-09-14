## Enter interactive mode and set up environment
```
srun --pty -A ic_ccb293 -p savio -t 02:00:00 bash -i
module load gcc/6.3.0
module load gsl
module load openblas
module load fftw
module load r/3.5.1
```

## Copy data files and executables to your home directory
```
cd
mkdir hgdp && cd hgdp
cp -r ~sandrahui/ccb293/hgdp/ .
```

## Smartpca
Run smpartpca on EUR populations
```
./bin/smartpca -p HGDP.smartpca.EUR.par &> HGDP.smartpca.EUR.log
```
### Parameter file HGDP.smartpca.par:
```
genotypename: HGDP.EUR.geno        # file with genotype information
snpname:      HGDP.EUR.snp         # file with snp information
indivname:    HGDP.EUR.ind         # file with individual information
poplistname:  HGDP.packed.EUR.list # list of pops to include in the run.
evecoutname:  HGDP.EUR.evec        # output file of eigenvectors.
evaloutname:  HGDP.EUR.eval        # output file of all eigenvalues
phylipname:   HGDP.EUR.phyl        # file with Fst values across populations
numoutevec:   10                   # number of PCs to output
numthreads:   1                    # if running interactively, use 1 only
```

#### Plot output
Plot the output in R using:
```
# start R 
R [enter]

# read output files
evecs <- read.table("HGDP.packed.EUR.evec")
colnames(evecs) <- c("sample", paste("PC", 1:10, sep=""), "pop")
evecs$pop <- factor(evecs$pop)

# plot
plot(x=evecs$PC1, y=evecs$PC2, col=factor(evecs$pop), xlab="PC1", ylab="PC2", pch=16)
legend("bottomright", legend=levels(evecs$pop), col=1:length(levels(evecs$pop)),pch=16)

# quit R
q() [enter]
```
Take a screenshot of the output or save it as an image file directly from R. To save it from R, run
```
png("HGDP.packed.EUR.PCA.png")
plot(x=evecs$PC1, y=evecs$PC2, col=factor(evecs$pop), xlab="PC1", ylab="PC2", pch=16)
legend("bottomright", legend=levels(evecs$pop), col=1:length(levels(evecs$pop)),pch=16)
dev.off()
```
This file can then be copied back to your computer using scp.


## ADMIXTURE
Run admixture for k=1..7 for all populations (already downsampled to 20k SNPs to decrease run time)
#### Command line: 
```
# Set k=1..7
for k in $(seq 1 7) ; do
  ./bin/admixture --cv HGDP.packed.20k.bed $k &> "HGDP.admix.20k.""$k"".log"
done
```
#### Input:
This program requires input in plink format (http://zzz.bwh.harvard.edu/plink/data.shtml). Note, K = number of clusters. Run the command for each value of K of interest (for example, 1..6)

#### Plot output
To read in the ADMIXTURE results:
```
# start R 
R [enter]


# quit R
q() [enter]
```
Take a screenshot of the resulting plot, or save it as an image file as above.

## ALDER
Run Alder for admixed populations Bedouin, Druze, and Palestinian (ref pops Yoruba and French)
```
./bin/alder -p HGDP.alder.Bedouin.par &> HGDP.alder.Bedouin.log
./bin/alder -p HGDP.alder.Palestinian.par &> HGDP.alder.Palestinian.log
./bin/alder -p HGDP.alder.Druze.par &> HGDP.alder.Druze.log
```
### Parameter file HGDP.alder.Bedouin.par (change admixpop for each population):
```
genotypename: HGDP.packed.geno # file with genotype information
snpname:      HGDP.packed.snp  # file with snp information
indivname:    HGDP.packed.ind  # file with individual information
admixpop:     Bedouin          # name of admixed population
refpops:      Yoruba;French    # name of reference (ancestral) population (seperated by semicolon).
checkmap:     NO
```


