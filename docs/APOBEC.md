
#	APOBEC


Analyzing C->T mutations


[A collection of perl and R scripts to format COSMIC mutational datasets and extract mutational signature information.](https://github.com/jakewendt/Mutation-Signatures)




##	VCF filtering

SNPs:
* QD  < 2.0         
* FS  >  60.0         
* MQ  <  40.0 	
* MQRankSum  <  -12.5 
* ReadPosRankSum  <  -8.0

Indels:
* QD  <  2.0         
* FS  >  200.0      
* ReadPosRankSum  <  -20.0 

note mapping quality filters are not recommended for indels https://qcb.ucla.edu/wp-content/uploads/sites/14/2017/08/gatkDay3.pdf




https://github.com/jakewendt/notebooks/blob/master/20190304-Mutation-Signatures.ipynb






https://github.com/mcjarvis/Mutation-Signatures-Including-APOBEC-in-Cancer-Cell-Lines-JNCI-CS-Supplementary-Scripts

https://academic.oup.com/jncics/article/2/1/pky002/4942295

https://github.com/raerose01/deconstructSigs

https://cancer.sanger.ac.uk/cell_lines



https://docs.google.com/document/d/1l4H4r6vZ8Y0B11p-RGfEbjWStoUl65211_FWE6Mz2kc/edit#


Continuation of Mutation Calling for Tumor-only Data

https://cancer.sanger.ac.uk/cosmic/signatures/SBS/


CCLS/20190205-vcf-tumor-normal

