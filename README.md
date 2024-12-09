Parvovirus Analysis
============================


1\. Platform Specific Setup 
============================

1.1 AWS Setup
-------------

Follow the separate AWS [setup guide](https://github.com/bioe131/lab-8-ella-deguzman/blob/main/aws_instructions.md), then return here to set up linuxbrew below.

1.2 Linuxbrew for AWS
---------------------

Make sure you are using a Debian or Ubuntu distribution. Then go ahead and install linuxbrew, using the instructions below:

switch to root with:

`sudo su -`
Then run:

`passwd ubuntu`
It is going to prompt :

`Enter new UNIX password:`

Set your password to something you can remember for later, or write down. A common password choice is simply ubuntu - not very secure at all, but AWS accounts themselves can be made fairly secure.

Exit root by typing `exit`. Note: it is important to exit root, because you do not want to accidentally run future commands with administrator privileges when that might be undesirable. The subsequent command in this case will fail if run from root.

Install brew using the bash script from <https://brew.sh/>. You will be prompted to set the password you made earlier.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After this is complete, add brew to your execution path:

```
echo >> /home/ubuntu/.bashrc

echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/ubuntu/.bashrc

eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

2\. Install necessary tools
===========================

2.1. Node.js
------------

Node.js is a cross-platform JavaScript runtime environment that will make is easy to run JBrowse2 command-line tools.

First, check whether Node.js is already installed by running the following. If node v20 is already installed, you can skip to the next step.

`node -v`

If Node.js is not installed, install it.

For AWS:
```
sudo apt install unzip
```
Then run 

```
# installs fnm (Fast Node Manager)
curl -fsSL https://fnm.vercel.app/install | bash
# activate fnm
source ~/.bashrc
# download and install Node.js
fnm use --install-if-missing 20
# verifies the right Node.js version is in the environment
node -v # should print `v20.18.0`
# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```

2.2. @jbrowse/cli
-----------------

Run the following command in your shell.
```
npm install -g @jbrowse/cli
```

2.3. System dependencies
------------------------

Install wget (if not already installed), apache2, samtools, and tabix.

wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP.

apache2 allows you to run a web server on your machine.

samtools and tabix, as we have learned earlier in the course, are tools for processing and indexing genome and genome annotation files.
```
sudo apt install wget apache2
brew install samtools htslib
```

3\. Apache server setup
=======================

3.1. Start the apache2 server
-----------------------------
```
sudo service apache2 start
```
3.2. Getting the host
---------------------

In your instance summary page, there should be an "auto-assigned IP address." Your web server can be accessed at `http://ipaddress`. You don't need to provide a port.

3.3. Access the web server
--------------------------

Open a browser and type the appropriate url into the address bar. You should then get to a page that says "It works!" If you have trouble accessing the server, you can try checking your firewall settings and disabling any VPNs or proxies to make sure traffic to localhost is allowed.

3.4. Verify apache2 server folder
---------------------------------

Apache2 web servers serve files from within a root directory. This is configurable in the httpd.conf configuration file, but you shouldn't have to change it (in fact, changing the conf file is not recommended unless you know what you are doing).

For a normal linux installation, the folder should be /var/www or /var/www/html, whereas when you install on macOS using brew it will likely be in /opt/homebrew/var/www (for M1) or /usr/local/var/www (for Intel). You can run brew --prefix to get the brew install location, and then from there it is in the var/www folder.

Verify that one of these folders exists (it should currently be empty, except possibly for an index file, but we will now populate it with JBrowse 2). If you have e.g. a www folder with no www/html folder, and your web server is showing the "It works!" message, you can assume that the www one is the root directory.

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the export if you restart your terminal session!

```
# be sure to replace the path with your actual true path!
# for example we used export APACHE_ROOT='/var/www/html'
export APACHE_ROOT='/path/to/rootdir'
```

3.5. Download JBrowse 2
-----------------------

First create a finalproj directory as a staging area. 
```
mkdir finalproj
cd finalproj
```

Next, download and copy over JBrowse 2 into the apache2 root dir, setting the owner to the current user with chown and printing out the version number. This version doesn't have to match the command-line jbrowse version, but it should be a version that makes sense.

```
jbrowse create output_folder
sudo mv output_folder $APACHE_ROOT/jbrowse2
sudo chown -R $(whoami) $APACHE_ROOT/jbrowse2
```

3.6. Test your jbrowse install
------------------------------

In your browser, now type in `http://yourhost/jbrowse2/`, where yourhost is either localhost or the IP address from earlier. Now you should see the words "It worked!" with a green box underneath saying "JBrowse 2 is installed." with some additional details.'


4\. Load and process test data
==============================

4.1. Download and process reference genome
------------------------------------------

Make sure you are in the temporary folder you created, then download the human, canine, rat, and porcine parvovirus genome in fasta format.

Human: 
```
export FASTA_ROOT=[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/)

wget $FASTA_ROOT/[GCF_000839645.1_ViralProj14090_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/GCF_000839645.1_ViralProj14090_genomic.fna.gz)
```
Canine :
```
export FASTA_ROOT=[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614/)

wget $FASTA_ROOT/GCF_000848925.1_ViralProj14614_genomic.fna.gz
```
Rat:
```
export FASTA_ROOT=[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1/)

wget $FASTA_ROOT/[GCA_031139815.1_ASM3113981v1_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1/GCA_031139815.1_ASM3113981v1_genomic.fna.gz)
```
Porcine
```
export FASTA_ROOT=<https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055>

wget $FASTA_ROOT/[GCF_000839485.1_ViralProj14055_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055/GCF_000839485.1_ViralProj14055_genomic.fna.gz)
```

4.2 Unzip reference genome
--------------------------

Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.

Human:
```
gunzip [GCF_000839645.1_ViralProj14090_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/GCF_000839645.1_ViralProj14090_genomic.fna.gz) 

mv [GCF_000839645.1_ViralProj14090_genomic.fna](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/GCF_000839645.1_ViralProj14090_genomic.fna.gz) hpv.fa

samtools faidx hpv.fa
```

Canine :
```
gunzip GCF_000848925.1_ViralProj14614_genomic.fna.gz   

mv GCF_000848925.1_ViralProj14614_genomic.fna cpv.fa

samtools faidx cpv.fa
```
Rat:
```
gunzip [GCA_031139815.1_ASM3113981v1_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1/GCA_031139815.1_ASM3113981v1_genomic.fna.gz)

mv [GCA_031139815.1_ASM3113981v1_genomic.fna](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1/GCA_031139815.1_ASM3113981v1_genomic.fna.gz) rpv.fa

samtools faidx rpv.fa
```
Porcine:
```
gunzip [GCF_000839485.1_ViralProj14055_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055/GCF_000839485.1_ViralProj14055_genomic.fna.gz)

mv [GCF_000839485.1_ViralProj14055_genomic.fna](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055/GCF_000839485.1_ViralProj14055_genomic.fna.gz) pcv.fa

samtools faidx pcv.fa
```
4.3. Load genome into jbrowse
-----------------------------
Human: 
```
jbrowse add-assembly hpv.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

Canine:
```
jbrowse add-assembly cpv.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

Rat:
```
jbrowse add-assembly rpv.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

Porcine:
```
jbrowse add-assembly ppv.fa --out $APACHE_ROOT/jbrowse2 --load copy
```

4.4. Download and process genome annotations
--------------------------------------------

Human:
```
export GFF_ROOT=[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/)

wget $GFF_ROOT/[GCF_000839645.1_ViralProj14090_genomic.gff.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/GCF_000839645.1_ViralProj14090_genomic.gff.gz)

gunzip GCF_000839645.1_ViralProj14090_genomic.gff.gz
```

Canine
```
export GFF_ROOT=[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614/)

wget $GFF_ROOT/[GCF_000848925.1_ViralProj14614_genomic.gff.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614/GCF_000848925.1_ViralProj14614_genomic.gff.gz)

gunzip [GCF_000848925.1_ViralProj14614_genomic.gff.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614/GCF_000848925.1_ViralProj14614_genomic.gff.gz)
```

Rat
```
export GFF_ROOT=[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1/)

[wget $GFF_ROOT/GCA_031139815.1_ASM3113981v1_genomic.gff.gz](http://.gff.gz)

gunzip [GCA_031139815.1_ASM3113981v1_genomic.gff.gz](http://.gff.gz)
```

Porcine:
```
export FASTA_ROOT=<https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055>

wget $FASTA_ROOT/[GCF_000839485.1_ViralProj14055_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055/GCF_000839485.1_ViralProj14055_genomic.fna.gz)

gunzip [GCF_000839485.1_ViralProj14055_genomic.fna.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055/GCF_000839485.1_ViralProj14055_genomic.fna.gz)
```

Use jbrowse to sort the annotations. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while making sure to preserve the header lines at the top of the file (which start with "#"). We then compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix. The tabix command outputs a file named genes.gff.gz.tbi in the same directory, and we then refer to "genes.gff.gz" as a "tabix indexed GFF3 file".

Human: 
```
jbrowse sort-gff GCF_000839645.1_ViralProj14090_genomic.gff > hpvgenes.gff
bgzip hpvgenes.gff
tabix hpvgenes.gff.gz
```

Canine:
```
jbrowse sort-gff GCF_000848925.1_ViralProj14614_genomic.gff > cpvgenes.gff
bgzip cpvgenes.gff
tabix cpvgenes.gff.gz
```

Rat:
```
jbrowse sort-gff GCA_031139815.1_ASM3113981v1_genomic.gff >  rpvgenes.gff
bgzip rpvgenes.gff
tabix rpvgenes.gff.gz
```

Porcine:
```
jbrowse sort-gff GCF_000839485.1_ViralProj14055_genomic.gff > ppv
bgzip ppvgenes.gff
tabix ppvgenes.gff.gz
```

4.5. Load annotation track into jbrowse
----------------------------------------
Human:
```
jbrowse add-track hpvgenes.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames hpv
```

Canine
```
jbrowse add-track cpvgenes.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames cpv
```

Rat
```
jbrowse add-track rpvgenes.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames rpv
```

Porcine
```
jbrowse add-track ppvgenes.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames ppv
```

4.6. Index for search-by-gene
-----------------------------

Run the "jbrowse text-index" command to allow users to search by gene name within JBrowse 2.
```
jbrowse text-index --out $APACHE_ROOT/jbrowse2
```

5\. Use your genome browser to explore a gene of interest
============================================================

5.1. Launch JBrowse2
---------------------

Open `http://yourhost/jbrowse2/` again in your web browser. There should now be several options in the main menu. Follow the guide in the "Launch the JBrowse 2 application and search for a gene in the linear genome view" section of https://currentprotocols.onlinelibrary.wiley.com/doi/10.1002/cpz1.1120 to navigate to the gene search and try browsing a few genes.


Multiple Sequence Alignment:
============================

1\. Add plugin to config.json file
==================================

Open config.json in AWS instance:
```
nano /var/www/html/jbrowse2/config.json
```

Add the MSA plugin by pasting the following JSON into the plugins section of the file:
```
"plugins": [

    {

      "name": "MsaView",

    "url": "https://unpkg.com/jbrowse-plugin-msaview/dist/jbrowse-plugin-msaview.umd.production.min.js"

    }

  ]
```

2\. Download and Process Reference Genome
=========================================

2.1 Set Up the Environment
---------------------------

Open a local terminal and navigate to the directory where you want to download genome files:
```
cd ~/Desktop/genomedata
```

Install required system dependencies:

```
brew install wget httpd samtools htslib
sudo brew services start httpd
```

2.2 Download and Prepare Genomes
---------------------------------

#### Human Parvovirus (HPV):
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/645/GCF_000839645.1_ViralProj14090/GCF_000839645.1_ViralProj14090_genomic.fna.gz
gunzip GCF_000839645.1_ViralProj14090_genomic.fna.gz
mv GCF_000839645.1_ViralProj14090_genomic.fna hpv.fa
samtools faidx hpv.fa
```

#### Canine Parvovirus (CPV):
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/925/GCF_000848925.1_ViralProj14614/GCF_000848925.1_ViralProj14614_genomic.fna.gz
gunzip GCF_000848925.1_ViralProj14614_genomic.fna.gz
mv GCF_000848925.1_ViralProj14614_genomic.fna cpv.fa
samtools faidx cpv.fa
```

#### Rat Parvovirus(RPV):
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/031/139/815/GCA_031139815.1_ASM3113981v1/GCA_031139815.1_ASM3113981v1_genomic.fna.gz
gunzip GCA_031139815.1_ASM3113981v1_genomic.fna.gz
mv GCA_031139815.1_ASM3113981v1_genomic.fna rpv.fa
samtools faidx rpv.fa
```

#### Porcine Parvovirus (PPV):
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/839/485/GCF_000839485.1_ViralProj14055/GCF_000839485.1_ViralProj14055_genomic.fna.gz
gunzip GCF_000839485.1_ViralProj14055_genomic.fna.gz
mv GCF_000839485.1_ViralProj14055_genomic.fna ppv.fa
samtools faidx ppv.fa
```

3\. Combine Sequences for Multiple Sequence Alignment
======================================================
```
cat hpv.fa cpv.fa rpv.fa ppv.fa> combined_sequences.fa
```
4\. Perform Multiple Sequence Alignment (MSA)
=============================================

Install and run MAFFT 
```
brew install mafft 
mafft --auto combined_sequences.fa > aligned_sequences.mfa
```

5\. Visualize the Alignment in JBrowse:
========================================

Go to `http://yourhost/jbrowse2/`, replacing yourhost with IP address.

Select an assembly then click Add > Multiple Sequence Alignment View. 

Click MSA file and then upload the `aligned_sequences.mfa` file. Or paste our link: `https://raw.githubusercontent.com/ella-deguzman/infection/refs/heads/main/aligned_sequences.mfa`

Now visualize the Multiple Sequence Alignment between human, canine, rat, and porcine parvovirus!


Ideogram View
=============

1\. Install temporarily on JBrowse:
========================================

Click on tools in the menu bar (should be in the top left of the screen).

Click on plugin store. 

On the right side of the screen, scroll down to where it shows the Ideogram Plug in. Press install.


2\. Install permanently on AWS instance:
========================================

2.1  Go to your AWS instance. Open config.json in AWS instance: 
---------------------------------------------------------------
```
nano /var/www/html/jbrowse2/config.json
```

2.2 Scroll down to plug-ins and add the new plug in permanently.
-----------------------------------------------------------------
```
{

      "name": "Ideogram",

      "url": "https://unpkg.com/jbrowse-plugin-ideogram/dist/jbrowse-plugin-ideogram.umd.production.min.js"

}
```
