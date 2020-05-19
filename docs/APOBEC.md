
#	APOBEC


[A collection of perl and R scripts to format COSMIC mutational datasets and extract mutational signature information.](https://github.com/jakewendt/Mutation-Signatures)



VCF filtering


SNPs:
• QD  < 2.0         
• FS  >  60.0         
• MQ  <  40.0 	
• MQRankSum  <  -12.5 
• ReadPosRankSum  <  -8.0


Indels:
• QD  <  2.0         
• FS  >  200.0      
• ReadPosRankSum  <  -20.0 
*note mapping quality filters are not recommended for indels
https://qcb.ucla.edu/wp-content/uploads/sites/14/2017/08/gatkDay3.pdf


