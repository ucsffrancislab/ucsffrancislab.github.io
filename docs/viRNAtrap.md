
#	viRNAtrap



```
module load WitteLab python3/3.9.1

cd ~/github/AuslanderLab/virnatrap/

python3 -m pip install --user .
```


```
mkdir output_contigs	#	REQUIRED TO EXIST

virnatrap-predict --num_threads 8 --input input_fastq/ --output output_contigs/ 

2023-10-17 07:40:31.827954: I tensorflow/tsl/cuda/cudart_stub.cc:28] Could not find cuda drivers on your machine, GPU will not be used.
2023-10-17 07:40:32.190095: E tensorflow/compiler/xla/stream_executor/cuda/cuda_dnn.cc:9342] Unable to register cuDNN factory: Attempting to register factory for plugin cuDNN when one has already been registered
2023-10-17 07:40:32.190204: E tensorflow/compiler/xla/stream_executor/cuda/cuda_fft.cc:609] Unable to register cuFFT factory: Attempting to register factory for plugin cuFFT when one has already been registered
2023-10-17 07:40:32.192031: E tensorflow/compiler/xla/stream_executor/cuda/cuda_blas.cc:1518] Unable to register cuBLAS factory: Attempting to register factory for plugin cuBLAS when one has already been registered
2023-10-17 07:40:32.385163: I tensorflow/tsl/cuda/cudart_stub.cc:28] Could not find cuda drivers on your machine, GPU will not be used.
2023-10-17 07:40:32.386893: I tensorflow/core/platform/cpu_feature_guard.cc:182] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
2023-10-17 07:40:36.291178: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Could not find TensorRT
Reading fastq at input_fastq/...
starting_prediction...
20183/20183 [==============================] - 45s 2ms/step





```
Took about 2 hours


Nothing terribly special in the output.

Only 3 real contigs and they blasted as Human





```
module load samtools
cd /francislab/data1/working/20230628-Costello/20231005-STAR/

mkdir -p virnatrap-test2/output_contigs
mkdir -p virnatrap-test2/input_fastq

for bam in out/p559SF13507-v1_S78.Aligned.sortedByCoord.out.bam out/p562SF13934-v1_S61.Aligned.sortedByCoord.out.bam out/p563SF13953-v1_S69.Aligned.sortedByCoord.out.bam out/P83SF12565-v1_S19.Aligned.sortedByCoord.out.bam ; do
echo $bam
base=$( basename ${bam} .Aligned.sortedByCoord.out.bam )
sbatch --job-name="${base}" --ntasks=4 --mem=30G --time=1-0 --wrap="samtools fastq -N -f4 -o ${PWD}/virnatrap-test2/input_fastq/${base}_unmapped.fastq ${bam}"
done

sbatch --job-name="virna" --ntasks=64 --mem=495G --time=10-0 --export=NONE --wrap="module load WitteLab python3/3.9.1 && virnatrap-predict --multi_proc 16 --num_threads 64 --input /francislab/data1/working/20230628-Costello/20231005-STAR/virnatrap-test2/input_fastq/ --output /francislab/data1/working/20230628-Costello/20231005-STAR/virnatrap-test2/output_contigs/"


```

Last step taking a very long time. Almost 24 hours now. Not sure which option would have sped it up. Up multi_proc? Only seems to be using 4 threads so 1 each.









