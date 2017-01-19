# EVE: Ensembl VEP on EC2

###Copyright 2016 Brian S. Cole, PhD
####License: GPLv3+

> Note: using EVE requires an AWS account and will incur expenses. Users must pay for the AWS resources utilized.

> Note: These examples work in the us-east-1 region.  The AMI image at the heart of the EVE CloudFormation template currently exists only in the us-east-1 region.  Efforts are underway to create a global mapping of AMI images in all regions around the world (except for GovCloud).


#Quick Start

###1. _Clone or Download teplate_

First, clone this repo or download it as a tarball and extract it to get the CloudFormation template (EVE.json).

###2. _Select stack parameters_

To launch a stack from the EVE CloudFormation template and connect to it, simply navigate to your AWS Console (the web console) and select "CloudFormation" from services.  Next, upload the EVE.json template using the on-screen menu item which says "Upload a template to S3".  Upload your EVE.json file.  Click "Next".  You'll then be asked to input a couple parameters.

First, add a name for your stack that has meaning for you and your project - this name will be displayed so you can keep track of which resources are devoted to this stack (e.g. "myStudy_EVE" would make sense).  

Next, if you plan to use the VEP cache (highly recommended!) and the plugins which require data on block storage (CADD, ExAC) select "Yes" on the "AttachVolume" parameter.  

For the "EveInstanceType" parameter, select the EC2 instance type that you'd like to use.  The compute-optimized instances are both faster and more cost-efficient than general-purpose instance types, but the m4.large instance type was found to be the most cost-efficient (and the slowest) in real-world benchmarking using an unimputed GWAS dataset with ~5,000 genotypes (see EVE manuscript).  Depending on your workload, you might select a c4.4xlarge instance for rapid execution or an m4.large instance for a no-plugin background annotation job in a non-time-critical project (if such a thing exists).

You can optionally encrypt your EBS volume (that will contain your data input and output) with the "EncryptVolume" parameter.  This will use EBS encryption, which is a subservice offered by AWS.  IMPORTANT NOTE: you are responsible for keeping data secure on AWS under the shared responsibility model.

Next, you can select default VPC and subnet unless you have your own you'd like to use (e.g. to integrate into other virtual appliances and/or cloud services).  Select your instance type from the dropdown menu.

You will need to select a keypair name for SSH access to your EC2 instance.  If you haven't generated one before, see [the AWS User Guide for EC2 key pairs.](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

Finally, select an EBS volume size that is at least 3x as big as your VCF file (uncompressed).  Allocate some extra space if you plan to do downstream analysis on the EVE EC2 instance itself, or to attach this EBS volume to a downstream appliance afterwards.

###3. _Launch stack_

Once all this is complete, click next and AWS will let you associate an IAM role with the stack if you desire.  You can also assign tags, for instance a name of a project used for internal record keeping.  Click next, then create.  The stack will launch and return public domain address for you to SSH in to under the "Output" tab.  This information will also be available from the EC2 console.

###4. _Upload data_

SSH connect to your instance using the key-pair file you used above.  Upload your data (e.g. SCP, aws s3 cp - but remember not to leave AWS credentials on the appliance if you take EBS snapshots for sharing).  Start annotation, for example:

###5. _Annotate variants_
```bash
variant_effect_predictor.pl --port 3337 --cache --vcf --no_stats --merged \
    --dir /data/.vep/ --dir_cache /data/.vep/ --dir_plugins /data/.vep/ --plugin GXA \
    --plugin GO --plugin CADD,/data/CADD/whole_genome_SNVs.tsv.gz,/data/CADD/InDels.tsv.gz \
    --plugin miRNA --plugin ExAC,/data/ExAC/ExAC.r0.3.1.sites.vep.vcf.gz \
    --fork $(nproc) --gmaf --buffer_size 50000 
    -i path/to/input_file.vcf -o path/to/output_file.vep.vcf
```
###6. _Clean up_

After annotation is complete, you can download your data (e.g. SFTP, aws s3 cp - but remember not to leave AWS credentials on the appliance if you take EBS snapshots for sharing).  Then you can shutdown the stack from the CloudFormation web console by selecting your EVE stack and clicking "Delete Stack" under options.  You can optionally take a snapshot of the EBS volumes for further use.





