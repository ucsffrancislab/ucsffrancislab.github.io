
#	ABySS

[ABySS](https://github.com/bcgsc/abyss)


Needs model compilers

I also installed the latest boost locally.


```
module load CBI scl-devtoolset

mkdir build
cd build
../configure --prefix=$HOME/.local 
make
make install
```

Why compile? The tar file includes binaries!





```

sbatch --job-name="abyss" --ntasks=16 --mem=120G --time=10-0 --export=NONE --wrap="~/.local/abyss-2.3.7/bin/abyss-pe --directory=/francislab/data1/working/20230628-Costello/20231005-STAR/abyss-test j=16 k=25 name=test B=10G in='/francislab/data1/working/20230628-Costello/20231005-STAR/out/p559SF13507-v1_S78.Aligned.sortedByCoord.out.unmapped.fasta.gz'"

```


