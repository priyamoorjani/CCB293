# CCB293
Pipeline for running smartpca, ADMIXTURE and ALDER

## Smartpca

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
```
#### Plot output

Plot the output in R using:
```
file <- read.table("outfile.evec")
plot(file$V2, file$V3, col=as.factor(file$V12), pch=16)
file = file[file$V12 %in% c("YRI", "CEU", "CHB","ASW"),]
```

## ADMIXTURE
#### Command line: 

```
./admixture --cv input.bed $K
```
#### Input:
This program requires input in plink format (http://zzz.bwh.harvard.edu/plink/data.shtml). Note, K = number of clusters. Run the command for each value of K of interest (for example, 1..6)

#### Plot output
To read in the ADMIXTURE results:
```
file1 = read.table("input.1.Q")
file2 = read.table("input.2.Q")
file3 = read.table("input.3.Q")
file4 = read.table("input.4.Q")
file5 = read.table("input.5.Q")
file6 = read.table("input.6.Q")
```
To plot the results:
```
par(mfrow=c(6,1), mar=c(1,4,1,1))
barplot(t(as.matrix(file1)), col=rainbow(1), border=NA)
barplot(t(as.matrix(file2)), col=rainbow(2), border=NA)
barplot(t(as.matrix(file3)), col=rainbow(3), border=NA)
barplot(t(as.matrix(file4)), col=rainbow(4), border=NA)
barplot(t(as.matrix(file5)), col=rainbow(5), border=NA)
par(mar=c(3,4,1,1))
x = barplot(t(as.matrix(file6)), col=rainbow(6), border=NA)
inds = c(rep('YRI', 3), rep('CEU', 3), rep('CHB', 10), rep('TSI', 10), rep('ASW', 10))
mtext(inds, 1, at=x, las=2)
```

## ALDER
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
