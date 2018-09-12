# CCB293
Pipeline for running smartpca, ADMIXTURE and ALDER

## Enter interactive mode and set up environment
```
srun --pty -A ic_ccb293 -p savio -t 01:00:00 bash -i
module load gcc/6.3.0
module load gsl
module load openblas
module load fftw
module load r/3.5.1
```

## Datafiles and executables:
```
/global/home/users/sandrahui/ccb293/    # data
/global/home/users/sandrahui/ccb293/bin # executables
~sandrahui/ccb293
```
## Smartpca

#### Documentation:
https://github.com/chrchang/eigensoft/blob/master/POPGEN/README

#### Command line: 
```
./smartpca -p <parfilename> 
```
#### Input:
This program requires data in EIGENSTRAT format (See https://reich.hms.harvard.edu/software/InputFileFormats). 
 
The program also requires a parameter file. See format below.
#### Parameter file arguments:
```
genotypename: infile.geno # file with genotype information
snpname: infile.snp       # file with snp information
indivname: infile.ind     # file with individual information   
evecoutname: outfile.evec # output file of eigenvectors.
evaloutname: outfile.eval # output file of all eigenvalues
poplistname: list         # list of pops to include in the run.
phylipname:  outfile.phyl # file with Fst values across populations 
numoutevec: 10            # number of PCs to output 
numoutliter: 0            # number of outlier inds to remove. 
```
#### Plot output

Plot the output in R using:
```
# read output files
evecs <- read.table("classEx.evec")
colnames(evecs) <- c("sample", paste("PC", 1:10, sep=""), "pop")

# filter down to only 4 populations
evecs <- evecs[evecs$pop %in% c("YRI", "CEU", "CHB", "ASW"),]
evecs$pop <- factor(evecs$pop)

#plot
plot(x=evecs$PC1, y=evecs$PC2, col=factor(evecs$pop), xlab="PC1", ylab="PC2", pch=16)
legend("bottomright", legend=levels(evecs$pop), col=1:length(levels(evecs$pop)),pch=16)
```

## ADMIXTURE

#### Documentation:
http://www.genetics.ucla.edu/software/admixture/admixture-manual.pdf

#### Command line: 
```
./admixture --cv input.bed $K
```
#### Input:
This program requires input in plink format (http://zzz.bwh.harvard.edu/plink/data.shtml). Note, K = number of clusters. Run the command for each value of K of interest (for example, 1..6)

#### Plot output
To read in the ADMIXTURE results:
```
# read output files
admix_k1 <- read.table("classEx.1.Q")
admix_k2 <- read.table("classEx.2.Q")
admix_k3 <- read.table("classEx.3.Q")
admix_k4 <- read.table("classEx.4.Q")
admix_k5 <- read.table("classEx.5.Q")
admix_k6 <- read.table("classEx.6.Q")

#plot
par(mfrow=c(6,1), mar=c(1,4,1,1))
barplot(t(as.matrix(admix_k1)), col=rainbow(1), border=NA)
barplot(t(as.matrix(admix_k2)), col=rainbow(2), border=NA)
barplot(t(as.matrix(admix_k3)), col=rainbow(3), border=NA)
barplot(t(as.matrix(admix_k4)), col=rainbow(4), border=NA)
barplot(t(as.matrix(admix_k5)), col=rainbow(5), border=NA)
par(mar=c(3,4,1,1))
x <- barplot(t(as.matrix(admix_k6)), col=rainbow(6), border=NA)

# add pop labels
inds <- c("ASW", rep("", 61), "CEU", rep("", 99), "CHB", rep("", 103), "TSI", rep("", 107), "YRI", rep("", 108))
mtext(inds, 1, at=x, las=1, adj=0)
```

## ALDER

#### Documentation:
https://github.com/priyamoorjani/CCB293/blob/master/ALDER.txt

#### Command line: 
```
./alder -p <parfilename> 
```
#### Input:
This program requires data in EIGENSTRAT format (See https://reich.hms.harvard.edu/software/InputFileFormats). The input geno, snp, and ind files must be consistent.
 
The program also requires a parameter file. See format below.
#### Parameter file arguments:
```
genotypename: infile.geno # file with genotype information
snpname: infile.snp       # file with snp information
indivname: infile.ind     # file with individual information   
admixpop:  ASW            # name of admixed population
refpops:   YRI;CEU        # name of reference (ancestral) population (seperated by semicolon).
```
#### Plot output

Included in the log file.
