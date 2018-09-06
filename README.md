# CCB293
Pipeline for running smartpca, ADMIXTURE and ALDER

## Smartpca

#### Command line: 
```
./smartpca -p <parfilename> 
```
#### Input:
This program requires data in EIGENSTRAT format (See https://reich.hms.harvard.edu/software/InputFileFormats). The input geno, snp, and ind files must be consistent.
 
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
### Plot output

Plot the output in R using:
```
file <- read.table("outfile.evec")
plot(file$V2, file$V3, col=as.factor(file$V12), pch=16)
file = file[file$V12 %in% c("YRI", "CEU", "CHB","ASW"),]
```

## ADMIXTURE
