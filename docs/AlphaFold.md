
#	AlphaFold


Install Docker and Singularity locally

Docker is very straight forward.

Singularity is tricky. Gotta install Homebrew, then a VM, currently lima, then singularity.



These instructions are a couple years old. Let's see if they hold up.

https://www.rbvi.ucsf.edu/chimerax/data/singularity-apr2022/afsingularity.html



---


##	How to make an AlphaFold Singularity Image

Tom Goddard

April 6, 2022

Here is how I made an AlphaFold singularity image for the UCSF Wynton cluster. The Wynton cluster does not allow running Docker images because of security issues, so we run it using a singularity image.

The AlphaFold github repository has scripts to build a Docker image for running AlphaFold. We build that, then convert it to a Singularity image.


###	Commands for building AlphaFold singularity image

The UCSF Wynton cluster does not support Docker and does not allow building singularity images since both require root privileges. So I do these steps on a desktop Ubuntu 20.04 system where I have root access.


Jake - For some reason I had to pull the nvidia docker image first. Just the first try though. 


https://github.com/google-deepmind/alphafold/issues/945

I created my own Dockerfile




I modified the lima singularity yaml file to mount the tmp dir as writable
```
- location: "/tmp/lima"
  writable: true
```


    
```
git clone git@github.com:deepmind/alphafold.git
cd alphafold


#	First time I had to pull this image first.
#	docker pull nvidia/cuda:12.2.2-cudnn8-runtime-ubuntu20.04

#	Not sure if its still version 220. After 2.3.2, but no real number. 2.3.3?
#	Not sure if this needs to be sudo either. 
#	ERROR: open /Users/jake/.docker/buildx/activity/desktop-linux: permission denied
#	fails without sudo if tried with sudo already

docker build -f ~/github/ucsffrancislab/genomics/docker/AlphaFold-2.3.3.Dockerfile -t alphafold233 .


#	Test
#
#	docker run -ti alphafold233
#	FATAL Flags parsing error:
#	  flag --fasta_paths=None: Flag --fasta_paths must have a value other than None.
#	  flag --output_dir=None: Flag --output_dir must have a value other than None.
#	  flag --data_dir=None: Flag --data_dir must have a value other than None.
#	  flag --uniref90_database_path=None: Flag --uniref90_database_path must have a value other than None.
#	  flag --mgnify_database_path=None: Flag --mgnify_database_path must have a value other than None.
#	  flag --template_mmcif_dir=None: Flag --template_mmcif_dir must have a value other than None.
#	  flag --max_template_date=None: Flag --max_template_date must have a value other than None.
#	  flag --obsolete_pdbs_path=None: Flag --obsolete_pdbs_path must have a value other than None.
#	  flag --use_gpu_relax=None: Flag --use_gpu_relax must have a value other than None.
#	Pass --helpshort or --helpfull to see help on flags.



#	Test

docker run --rm --entrypoint bash alphafold233 /app/run_alphafold_test.sh
#OpenBLAS WARNING - could not determine the L2 cache size on this system, assuming 256k
#OpenBLAS WARNING - could not determine the L2 cache size on this system, assuming 256k



#docker run -v /tmp:/tmp -it --rm --entrypoint bash alphafold233 /app/run_docker.sh  --data_dir=/tmp/lima/  --max_template_date=2020-05-14  --model_preset=monomer  --fasta_paths=/tmp/lima/SPELLARDPYGPAVDIWSAGIVLFEMATGQ.faa  --output_dir=/tmp/lima/


#		/app/run_docker.sh  --data_dir=/tmp/lima/  --max_template_date=2020-05-14  --model_preset=monomer  --fasta_paths=/tmp/lima/SPELLARDPYGPAVDIWSAGIVLFEMATGQ.faa  --output_dir=/tmp/lima/

#	python3 docker/run_docker.py --data_dir=/tmp/lima/  --max_template_date=2020-05-14  --model_preset=monomer  --fasta_paths=/tmp/lima/SPELLARDPYGPAVDIWSAGIVLFEMATGQ.faa  --output_dir=/tmp/lima/


python3 run_alphafold.py
  --bfd_database_path=/tmp/lima/ \
  --uniref30_database_path=/tmp/lima/ \
  --pdb70_database_path=/tmp/lima/ \
  --uniref90_database_path=/tmp/lima/uniref90.fasta \
  --mgnify_database_path=/tmp/lima/mgy_clusters_2022_05.fa \
  --template_mmcif_dir=/tmp/lima/ \
  --obsolete_pdbs_path=/tmp/lima/obsolete.dat \
  --use_gpu_relax \
  --data_dir=/tmp/lima/ \
  --max_template_date=2020-05-14 \
  --model_preset=monomer \
  --fasta_paths=/tmp/lima/SPELLARDPYGPAVDIWSAGIVLFEMATGQ.faa \
  --output_dir=/tmp/lima/




#	DON'T SAVE THIS TO THIS DIR OR NEXT TIME THE IMAGE WILL BE HUGER!
docker save alphafold233 -o ~/alphafold233_docker.tar



#	Create a VM if not existant already
#	limactl start ./singularity-ce.yml

cd ~
limactl shell singularity-ce

#	where is the HOME DIR so that files can be shared with outside the VM?
# /tmp/lima is set to writable in the yml




#singularity build alphafold233.sif docker-archive://alphafold233_docker.tar
#FATAL:   While performing build: while creating SIF: while creating container: open /Users/jake/github/google-deepmind/alphafold/alphafold233.sif: read-only file system


singularity build /tmp/lima/alphafold233.sif docker-archive://alphafold233_docker.tar


#	Test
singularity exec /tmp/lima/alphafold233.sif /app/run_alphafold_test.sh
#/sbin/ldconfig.real: Can't create temporary cache file /etc/ld.so.cache~: Read-only file system
#OpenBLAS WARNING - could not determine the L2 cache size on this system, assuming 256k
#OpenBLAS WARNING - could not determine the L2 cache size on this system, assuming 256k


singularity exec --bind /francislab,/scratch \
>  /francislab/data1/refs/alphafold/alphafold233.sif \
>  /app/run_alphafold.sh \
>  --bfd_database_path=/francislab/data1/refs/alphafold/databases/bfd/ \
>  --uniref30_database_path=/francislab/data1/refs/alphafold/databases/uniref30/ \
>  --pdb70_database_path=/francislab/data1/refs/alphafold/databases/pdb70/ \
>  --uniref90_database_path=/francislab/data1/refs/alphafold/databases/uniref90/uniref90.fasta \
>  --mgnify_database_path=/francislab/data1/refs/alphafold/databases/mgnify/mgy_clusters_2022_05.fa \
>  --template_mmcif_dir=/francislab/data1/refs/alphafold/databases/pdb_mmcif/ \
>  --obsolete_pdbs_path=/francislab/data1/refs/alphafold/databases/pdb_mmcif/obsolete.dat \
>  --use_gpu_relax \
>  --data_dir=/francislab/data1/refs/alphafold/databases/ \
>  --max_template_date=2020-05-14 \
>  --model_preset=monomer \
>  --fasta_paths=/francislab/data1/refs/SPELLARDPYGPAVDIWSAGIVLFEMATGQ.faa \
>  --output_dir=/francislab/data1/refs/alphafold/





#rsync -av alphafold233.sif plato.cgl.ucsf.edu:alphafold_singularity
#Script to run AlphaFold singularity image on Wynton
```





The AlphaFold github code repository contains a run_docker.py script for running the AlphaFold Docker image. I wrote a similar script run_alphafold220.py for running the AlphaFold Singularity image.

The script is designed for submitting jobs queued on the Wynton cluster using the Sun Grid Engine (SGE) queueing system. The comment lines at the top of the script give default SGE qsub command options to run in the GPU queue for up to 48 hours on an A40 or A100 GPU.

My script was written starting with the run_docker.py and replacing the docker image invocation with the equivalent singularity image invocation. I made many other changes to the AlphaFold run_docker.py script to improve ease of use:

Remove Google absl. The run_docker.py script requires Googles utility Python module absl is installed for argument parsing. I did not want to require Wynton users to install that Python module. So I adapted the argument parsing to use the standard Python argparse module.

Database file location. Changed the "data_dir" option to the location of the AlphaFold database files on Wynton and made specifying it optional.
Preset for PAE. Changed the default preset from monomer to monomer_ptm so PAE error estimates are produced.
Output directory. Changed the default output directory to "output", was /tmp/alphafold.
PDB template dates. Changed max_template_date to default to include all available PDB templates and made specifying it optional.
Number of multimer models. Compute by default just 1 model for each of the 5 alphafold multimer neural nets. AlphaFold 2.2.0 run_docker.py uses 5 which computes 25 total models.
Submitting an AlphaFold job on Wynton
Here is how to queue an AlphaFold jobs on Wynton.

      cd /wynton/home/ferrin/goddard/alphafold_singularity
      qsub run_alphafold220.py --fasta_paths=seq_7p8x_A.fasta
    
with FASTA file seq_78px_A.fasta as given here

>7P8X_1|Chain A|Leucotoxin LukEv|Staphylococcus aureus (1280)
MSVGLIAPLASPIQESRANTNIENIGDGAEVIKRTEDVSSKKWGVTQNVQFDFVKDKKYNKDALIVKMQGFINSRTSFSDVKGSGYELTKRMIWPFQYNIGLTTKDPNVSLINYLPKNKIETTDVGQTLGYNIGGNFQSAPSIGGNGSFNYSKTISYTQKSYVSEVDKQNSKSVKWGVKANEFVTPDGKKSAHDRYLFVQSPNGPTGSAREYFAPDNQLPPLVQSGFNPSFITTLSHEKGSSDTSEFEISYGRNLDITYATLFPRTGIYAERKHNAFVNRNFVVRYEVNWKTHEIKVKGHNKHHHHHH
And here is an example running a multimer prediction with two proteins

      qsub run_alphafold220.py --fasta_paths=seq_6z03.fasta --model_preset=multimer
    
with the two sequences in FASTA file seq_6z03.fasta containing

>6Z03_1|Chains A|DNA topoisomerase I|Caldiarchaeum subterraneum (311458)
MVKWRTLVHNGVALPPPYQPKGLSIKIRGETVKLDPLQEEMAYAWALKKDTPYVQDPVFQKNFLTDFLKTFNGRFQDVTINEIDFSEVYEYVERERQLKADKEYRKKISAERKRLREELKARYGWAEMDGKRFEIANWMVEPPGIFMGRGNHPLRGRWKPRVYEEDITLNLGEDAPVPPGNWGQIVHDHDSMWLARWDDKLTGKEKYVWLSDTADIKQKRDKSKYDKAEMLENHIDRVREKIFKGLRSKEPKMREIALACYLIDRLAMRVGDEKDPDEADTVGATTLRVEHVKLLEDRIEFDFLGKDSVRWQKSIDLRNEPPEVRQVFEELLEGKKEGDQIFQNINSRHVNRFLGKIVKGLTAKVFRTYIATKIVKDFLAAIPREKVTSQEKFIYYAKLANLKAAEALNHKRAPPKNWEQSIQKKEERVKKLMQQLREAESEKKKARIAERLEKAELNLDLAVKVRDYNLATSLRNYIDPRVYKAWGRYTGYEWRKIYTASLLRKFKWVEKASVKHVLQYFAEKLAKDVDKGMQVKAAV
>6Z03_2|Chains B|DNA topoisomerase I|Caldiarchaeum subterraneum (311458)
MVKWRTLVHNGVALPPPYQPKGLSIKIRGETVKLDPLQEEMAYAWALKKDTPYVQDPVFQKNFLTDFLKTFNGRFQDVTINEIDFSEVYEYVERERQLKADKEYRKKISAERKRLREELKARYGWAEMDGKRFEIANWMVEPPGIFMGRGNHPLRGRWKPRVYEEDITLNLGEDAPVPPGNWGQIVHDHDSMWLARWDDKLTGKEKYVWLSDTADIKQKRDKSKYDKAEMLENHIDRVREKIFKGLRSKEPKMREIALACYLIDRLAMRVGDEKDPDEADTVGATTLRVEHVKLLEDRIEFDFLGKDSVRWQKSIDLRNEPPEVRQVFEELLEGKKEGDQIFQNINSRHVNRFLGKIVKGLTAKVFRTYIATKIVKDFLAAIPREKVTSQEKFIYYAKLANLKAAEALNHKRAPPKNWEQSIQKKEERVKKLMQQLREAESEKKKARIAERLEKAELNLDLAVKVRDYNLATSLRNYIDPRVYKAWGRYTGYEWRKIYTASLLRKFKWVEKASVKHVLQYFAEKLAKDVDKGMQVKAAV
Running AlphaFold using Docker on Desktop
We run AlphaFold on an Ubuntu 20.04 machine in the lab with Nvidia RTX 3090 graphics minsky.cgl.ucsf.edu. For simpler use we modify the AlphaFold run_docker.py script to provide better default values as follows af220_minsky.py

output_dir. Default changed to current directory instead of /tmp/alphafold.
data_dir. Default changed to sequence databases location on minsky instead of being a required option.
docker_image_name. Default changed to alphafold220 instead of alphafold. Use version suffix so older alphafold versions can be run (by other scripts).
max_template_date. Default to year 2100 instead of being a required option.
model_preset. Default to monomer_ptm instead of monomer so PAE error estimates are produced.
num_multimer_predictions_per_model. Default changed to 1 instead of 5.






