
#	Nextflow



Notes on using Nextflow


Example run

```bash
module load openjdk; nextflow run ~/github/genepi/imputationserver2/main.nf -config ${PWD}/imputation_and_pgs-onco_1347.config"
```


Create a local config file like 'nextflow.config'

Might get weird errors like wrong partition

```bash
sbatch: error: Batch job submission failed: Invalid account or account/partition combination specified
```

```bash
process {
	executor = 'slurm' // --profile slurm
	queue = 'francislab'	// ? 'QueueName'  // replace with your Queue name
	time = '14d' // Set a time limit of 5 days for this process.
	queueSize = 20 // not sure that this makes much difference

	mailUser = 'George.Wendt@ucsf.edu' // Replace with your email address
	mailType = 'FAIL' // Or other types like BEGIN, END, REQUEUE, ALL

	clusterOptions = ''  //  drop this -A overbaugh_j the phippery example contains
}
```



What apps / pipelines are using it?

phippery
OHDSI/Broadsea
genepi imputation server
Gecko



