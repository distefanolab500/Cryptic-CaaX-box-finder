Cryptic CaaX box finder
Overview
Analysis on various proteomics data revealed the statistically significant enrichment of various proteins that were not known to be previously prenylated. Moreover, these proteins were not known to have appropriate consensus sequences to be prenylated by any of the three common prenylation enzymes. It was our hypothesis that there could be proteases that could reveal consensus sequences in certain proteins which would then allow the peptide to be prenylated.

The Cryptic CaaX box finder is a Python based algorithm that identifies potential Caa(a)X-box motifs in proteomics data. CaaX boxes are protein prenylation signals (where C = cysteine, a = aliphatic amino acid, X = any amino acid) that are substrates for post-translational modification by protein prenyltransferases.

This notebook integrates proteomics data with UniProt sequence data, prenylation prediction algorithms (PrePS), and N-terminal mass spectrometry (TopFind) data to identify potentially novel prenylated proteins.


Workflow Overview
Step 1: Data preparation
•	Input files required:
o	[filename].csv - Proteomics peptides file
o	[prenyldata_file].xlsx - Volcano plot file with statistically significant, positively enriched proteins
o	Total prenylation list – [prenylated_file].csv - Known bonafide prenylated proteins (used as filter)
•	Data cleaning operations:
o	Removes rows with missing UniProt IDs or end positions
o	Handles multiple UniProt IDs per peptide (selects first, this is arbitrary)
o	Filters volcano plot data for significantly enriched proteins only (marked with "+")
o	Excludes known bonafide prenylated proteins to focus on novel candidates
Step 2: Sequence Retrieval from UniProt
•	Submits significantly enriched proteins to UniProt ID mapping API
•	Retrieves protein sequences in FASTA format
•	Saves results to {filename}_{prenyldata_file}_uniprot_results.fasta
•	Uses Job ID polling system to monitor batch request completion
Step 3: Tryptic Cleavage Site Mapping
•	Extracts end positions of all tryptic cleavage sites for each protein
•	Identifies the last tryptic cleavage site for each protein
•	This position is used as the reference point for C-terminal prenylation detection
Step 4: CaaX Box Detection
•	Generates all possible combinations of CaaX motifs 
•	Scans protein sequences downstream of the last tryptic cleavage site
•	Identifies proteins containing CaaX motifs in their C-terminal region
•	Extracts and trims sequences to include only the region up to and including the CaaX box
Step 5: Sequence Length Normalization
•	The Prenylation Prediction Suite (PrePS) requires sequences ≥15 amino acids
•	Pads shorter sequences with lysine residues (KKK...) to meet this requirement
Step 6: Prenylation Scoring
•	Submits sequences to Prenylation Prediction Suite (PrePS)
•	PrePS predicts prenylation scores for two enzyme classes: Farnesyltransferase and Geranylgernayltransferase I
•	Scores range from negative (unlikely) to positive (likely prenylation)
Step 7: Score Filtering
•	Retains only proteins with at least one prenylation score > -2.0
•	Removes duplicate entries per protein
•	Outputs filtered results to {filename}_{prenyldata_file}_CaaX_final.csv
Step 8: N-Terminal Analysis (Optional)
•	Queries TopFIND database for experimentally identified N-termini
•	Retrieves C-terminal sequences identified by mass spectrometry
•	Cross-references TopFIND C-termini with CaaX box regions
•	Identifies cases where experimentally detected termini match CaaX boxes
•	Saves results CCBF_final.xlsx

Input Files
File Name, Format,	Description
[filename].csv	CSV	Proteomics peptides with protein IDs and end positions
[prenyldate_file].xlsx	XLSX	Volcano plot data; proteins marked with "+" indicate significant enrichment
Total prenylation list -[prenylated_file].csv	CSV	Known prenylated proteins (used for exclusion filtering)

Note: Update file paths in the first code cell to match your local files.

Output Files
File Name, Format,	Description
{filename}_{prenyldata_file}_uniprot_results.fasta	FASTA	Protein sequences retrieved from UniProt
{filename}_{prenyldata_file}_CaaX_final.csv	CSV	Main output - predicted prenylated proteins with scores
[CCBF_final].xlsx	XLSX	N-terminus data from TopFIND for validated proteins
Main Output Columns (CaaX_final.csv)
•	Protein IDs - UniProt accession
•	FTase scores - Farnesyltransferase prenylation scores
•	GGTase scores - Geranylgeranyl transferase I scores
•	sequences - Detected CaaX box motif(s)



Dependencies

•	pandas              
•	BioPython    
•	requests            
•	BeautifulSoup       
•	re                  
•	time                
•	collections         
•	numpy       
        
External APIs Used

1.	UniProt ID Mapping - https://rest.uniprot.org/idmapping/
2.	Prenylation Prediction Suite - https://mendel.imp.ac.at/PrePS/
3.	TopFIND Database - https://topfind.clip.msl.ubc.ca/nterms/

Usage

1.	Prepare input files with the filenames specified in the first code cell
2.	Update file paths as needed for your local environment
3.	Run cells sequentially - the pipeline depends on outputs from previous steps
4.	Monitor API calls - includes progress messages for UniProt, PrePS, and TopFIND queries
5.	Check output files - review scores in the final CSV and N-terminal data in Excel

Key Parameters & Thresholds
Parameter	Value	Purpose
Prenylation Score Threshold	> -2.0	Minimum score to retain proteins
Minimum Sequence Length	15 aa	PrePS requirement
CaaX Motif Position	C-terminal	After last tryptic cleavage site
Aliphatic Amino Acids	A,G,V,L,M,I + others	Valid 2nd position in CaaX
API Polling Interval	5 seconds	Wait between status checks
TopFIND Request Delay	1 second	Avoid rate limiting
 
Important Notes
Data Assumptions
•	UniProt IDs with semicolons - Only the first ID is retained (arbitrary choice)
•	Multiple CaaX boxes - Proteins can have multiple CaaX motifs, all are reported
•	Known prenylated proteins - Automatically excluded from analysis to focus on novel candidates
API Rate Limiting
•	The notebook includes delays and polling to handle API rate limits gracefully
•	Large batch submissions to PrePS and TopFind may take several minutes
Cryptic CaaaX box finder
•	Many instances of CaaaX boxes being prenylated have also been reported. The cryptic CaaaX box finder works exactly as described above expect that the algorithm removes the third aliphatic amino acid residue for scoring purposes.









