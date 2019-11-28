# SCOUT USER EXAMPLE
## Installation and setup for docker version

Clone the latest Scout version from Github, and the docker files needed:  
```
git clone https://github.com/Clinical-Genomics/scout scout-forked
git clone https://github.com/gmc-norr/scout-docker.git
```


Scout requires MongoDB to run. Easiest is to use the official MongoDB docker image and define usage in the docker-compose file. The docker compose file handles all the docker images associated with scout, and mounts needed folders. It also contains API-keys and any potential certificate files and config files. The docker compose file is not included in the git repo, but should be places in the `scout-docker` folder.

Before setting up the database, make sure to have the following folders on the same level as `scout-forked` and `scout-docker`:
- `database/`    # where the MongoDB data is stored
- `sampledata/`  # from where the data is read (if data is located elsewhere, change in docker-compose file)
- `vault/`       # contains the config file and all certificate files
- `panels/`      # contains all panels to be added to Scout


Setup the database and run Scout using the `Makefile`:

```
make build    # Builds all images listed in the Makefile
make setup    # Initializes the database, downloads data, initialize loqusdb connection, and adds institute and admin
make run      # Starts the scout application using gunicorn
make stop     # Stops the scout application
```


## Annotation of VCF files for use in Scout

Scout can work with both single sample VCF files, and family VCF files consisting of several related samples. Any caller should?? work, but make sure that all multiallelic variants have been split into separate variants.

### Example annotations:

##### VEP
```
vep \
--fork 4 \
--distance 5000 \
--buffer_size 20000 \
--assembly GRCh37 \
--fasta <fasta> \
--dir_cache <cache-dir> \
--format vcf \
--vcf \
--chr 1 \
--dir_plugins <plugin-dir> \
--plugin ExACpLI,<file.txt> \
--plugin LoFtool \
--plugin MaxEntScan,<input> \
--custom /home/proj/production/rare-disease/references/references_7.1/grch37_genomics_super_dups_-20181009.bed.gz,genomic_superdups_frac_match,bed,overlap,0 \
--appris \
--biotype \
--cache \
--canonical \
--ccds \
--domains \
--exclude_predicted \
--force_overwrite \
--hgvs \
--humdiv \
--no_progress \
--no_stats \
--numbers \
--merged \
--polyphen p \
--protein \
--offline \
--regulatory \
--sift p \
--symbol \
--tsl \
--uniprot \
--input_file <input.vcf> \
--output_file <output.vcf>
```

Annotations present after VEP:
- HGNC id (central for Scout)
- SYMBOL
- Transcripts
- Polyphen score
- Sift score
- more....

This part is still not working, try downloading all VEP databases again. Also look up custom and --chr 1. Try adding CADD annotations here also.

##### Genmod
Genmod is used for 3 different annotations:  
_models_: Checks pattern of inheritance that are followed in a VCF file.  
_scores_: Scores each variant using its annotations and a predefined algorithm  
_compound_: Scores compound variants using their Rank score  

`genmod models` requires a `pedfile` defining the relationships of the samples in the VCF file to work. An example of a `pedfile` can be found [here]('https://github.com/moonso/genmod/blob/master/examples/recessive_trio.ped'). Make sure that the `Family ID` used in this file matches the `family` id used in the config file later, otherwise Scout does not recognize the annotations.

To score all variants and give them a Rank score, genmod needs a file defining which annotations to use, where they are, and what scores to give them. An example of such file can be found [here]('https://github.com/moonso/genmod/blob/master/examples/score_test.ini').

Run the annotations with:
```
cat <input.vcf> |
genmod models - --family_file <family.ped> --vep | \
genmod score - -c <rank_score.model.ini> | \
genmod compound - --vep > \
<output.vcf>
```

Annotations present after genmod:  
- Genetic model  
- Rank Score  
- From compound??  

---
__NOTE:__     
Variant can be present in different genes depending on different splicing of transcripts. Some Rank scores can then be higher for a variant because of annotations from one gene, but the Rank score from the other gene would be lower. If you are only interested in certain genes in a gene list, it is better to remove all other genes not present in the gene list before doing the genmod annotations.

---

## Handling users in Scout

Users can be added with the help of the script `user-handler.py`. That script can both add and remove users from the database by starting different docker images.

__Add users:__  
```
python3 user-handler.py --add --name <name> --instid <instid> --adid <adid> --usermail <usermail>
```
If user should also be admin the flag `--admin` should be added. `--name` can in this setup not contain spaces due to docker specific issues. `--adid` is the AD/LDAP username used for authentication against the AD, and this setup is specific for that type of login.

__Remove users:__  
```
python3 user-handler.py --remove --usermail <usermail>
```

## Loading data into Scout

### Prepare and load gene panels

### Prepare config file

### Load data
