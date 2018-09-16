## Enter interactive mode and set up environment
```
srun --pty -A ic_ccb293 -p savio -t 01:00:00 bash -i
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
cp -rpL ~sandrahui/ccb293/hgdp/* .
```

## Smartpca
Run smpartpca on EUR populations (takes ~1m17s)
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
evecs <- read.table("HGDP.EUR.evec")
colnames(evecs) <- c("sample", paste("PC", 1:10, sep=""), "pop")
evecs$pop <- factor(evecs$pop)

# plot
# png("HGDP.packed.EUR.PCA.png") # uncomment to save as png
# pdf("HGDP.packed.EUR.PCA.pdf") # uncomment to save as pdf
plot(x=evecs$PC1, y=evecs$PC2, col=factor(evecs$pop), xlab="PC1", ylab="PC2", pch=16)
legend("bottomright", legend=levels(evecs$pop), col=1:length(levels(evecs$pop)),pch=16)
# dev.off() # uncomment to save image

# quit R
q() [enter]
```
Take a screenshot of the output or save it as an image file directly from R (uncomment appropriate lines above). This file can then be copied back to your computer using scp/sftp. [Instructions can be found on the BRC website](http://research-it.berkeley.edu/services/high-performance-computing/transferring-data)


## ADMIXTURE
Run admixture for k=1..7 for all populations (already downsampled to 20k SNPs to decrease run time; takes ~18m30s in total)
```
# Set k=1..7
for k in $(seq 1 7) ; do
  ./bin/admixture --cv HGDP.packed.20k.bed $k &> "HGDP.admix.20k.""$k"".log"
done
```

#### Plot output
Plot the output in R using:
```
# start R 
R [enter]

# get sample ordering according to continent and population
continents <- read.table("HGDP.packed.ind.continents", header=F, sep="\t", stringsAsFactors=F)
colnames(continents) <- c("pop", "continent")
indPop <- read.table("HGDP.packed.ind", header=F, stringsAsFactors=F)
colnames(indPop) <- c("ind", "sex", "pop")
indPop$continent <- sapply(indPop$pop, FUN=function(pop) {continents[continents$pop == pop, "continent"]})
sampOrdering <- order(indPop$continent, indPop$pop)
runLengths <- rle(indPop$pop[sampOrdering])
runLengths <- data.frame(pop=unname(unlist(runLengths["values"])), count=unname(unlist(runLengths["lengths"])))
popLines <- unlist(apply(runLengths, 1, FUN=function(row){c(row["pop"], rep("", as.numeric(row["count"])-1))}))
inds <- unlist(apply(runLengths, 1, FUN=function(row){c(rep("", floor((as.numeric(row["count"])-1)/2)), row["pop"], rep("", ceiling((as.numeric(row["count"])-1)/2)))}))

# read in admixture results and plot
kVals <- 1:7
# pdf("HGDP.packed.admixPlot.pdf") # uncomment to save as pdf
# png("HGDP.packed.admixPlot.png") # uncomment to save as png; may include plotting artifacts
par(mfrow=c(8,1), mar=c(1,4,1,1))
invisible(sapply(kVals, FUN=function(k) {
  admix <- read.table(paste("HGDP.packed.20k.", k, ".Q", sep=""))
  x <<- barplot(t(as.matrix(admix[sampOrdering,])), col=rainbow(k), xaxt='n', border=NA)
  abline(v=x[popLines != ""], lwd=2)
}))
mtext(inds, 1, at=x+5, las=2, adj=1, cex=.6)
# dev.off() # uncomment to save image

# quit R
q() [enter]
```
Take a screenshot of the resulting plot or save it as an image file as above.


## ALDER
Run Alder for admixed populations Bedouin, Druze, and Palestinian (ref pops Yoruba and French; takes ~2m30s in total)
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
