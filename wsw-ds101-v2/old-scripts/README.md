# Workshop Worker - Hortonworks Data Science 101

This repo houses some Ansible scripts that will deploy a CloudBreak workshop, setup new (easier) R53 subdomains, and even do the thing with the Certbot jazz.
## Requirements
  - Ansible 2.5+ on your worker...master...whatever you're calling the laptop you're running Ansible off of.
  - AWS Account with limitations removed (see below)
  - FQDN available in AWS Route 53
  - boto/boto3
  - Magic

## New Features!
  - It's fast!
  - It works!
  - It's REALLY fast!
## Todo
  - Variable-ize the CBD User/Password
  - Clean up comments
  - Add debug tags

## Instructions
  1) Clone repo down to local machine you have Ansible installed on
  2) Make sure you have your AWS Access/Secret keys exported via ~/.aws/credentials (**IMPORTANT**), boto setup, etc
  3) Copy the *'wsw_ds101-vars-example.yml'* file to *'wsw_ds101-vars.yml'*.
  4) Modify the *'wsw_ds101-vars.yml'* file, specifically noting to change
      * **certbot_email:** Your email
      * **seat_count:** How many workshop students/seats/users
      * **aws_region**
      * **workshop_prefix**
      * **workshop_domain**
  The rest you can leave as defaults, or change them to suit your needs.  The variable file is very well commented.
  5) There are 4 parts of this script
      * ***ansible-playbook ./1_wsw_ds101-createStacks.yml*** - This first command will build the frame of the Route 53 infrastructure, and deploy the stacks.  The cloudformation module is run in asyncronous mode to prevent having to wait for each stack to complete before starting the build of the next.  Goes from hours of building to about 20 minutes.  Once this command is done, you'll be able to see the stacks building in AWS CloudFormation.  Run this command, then wait about 15 minutes, or until they're marked as 'CREATE_COMPLETE' in CloudFormation.
      * ***ansible-playbook ./1_wsw_ds101-createStacks.yml --tags build*** - The following command will loop with an included task file to build out the remaining DNS records needed for the custom A records, *cbd-{seat_number}.{workshop_prefix}.{workshop_domain}, or maybe cbd-12.ds101.fiercesw.network*.  This step will also build the needed files locally that we'll use in the tear-down process.  Run this command, then wait about 10 minutes to have the new DNS records propogate.
       Oh, and why files?  Because again, AWS' cloudfoundation and especially route53 modules, are dumb and POORLY documented.  Like this is better than their Ansible module documentation.  Sad.
      * ***ansible-playbook -i {GENERATED_INVENTORY_FILE} ./2_wsw_ds101-configure.yml*** - Technically the third command, but second file.  This will do a few things such as install EPEL, CertBot, build an SSL certificate for the friendly FQDN we just put in Route53 in the previous step.  Does all the needed start/stopping of CBD.  Also, don't worry, if you pay attention to the output of the previous command, you'll have the generated inventory file listed for you with the full command.  Or it'll be in the *'.{{ workshop_prefix }}'* folder by default.
      * ***ansible-playbook ./3_wsw_ds101-dismantle.yml*** - This command will delete the stacks, DNS entries, EC2 keys, and local files.

Front to back, you can set up a 50 user workshop cluster in less than 30 minutes, update it, set up friendly FQDNs, and tear it down.  Nifty, eh?
## ***...GOTCHAS...***
So there are a few things though, and they're documented along the way if you just pay attention to the prompts and responses.
 - Run the first command, then wait about 15 minutes.  The first command will spin up the stacks which take about 15 minutes or so.  Since we're running them all in batch asyncronously it only takes 15 minutes for as many as we need BUT we have to wait after we start the stack building.  Go for a walk or something, these screens are bad for us.

## FAQS
 - **Couldn't you have made this a role, or even better, a bunch of roles?**  Meh, probably.  But AWS's cloudformation modules are really weird so that was kind of hard to do.  If you think it's do-able, by all means go for it!  Fork the repo, and get to collaborating!
 - **Wow, this is fast!**  Funny, because at first it was all in a set of roles and then I had to wait 15 minutes per Stack to finish spinning up...not fun.  Batch processing FTW.

## AWS Limitations
If you attempt to spin up more than half a dozen or so of these clusters, you'll notice that they'll rollback quickly.
That's because AWS has resource limits.  You'll need to increase your EIP, VPC, and EC2 - m4.large/m4.xlarge limits up; request that through the Support panel well before you need to spin up 100 seats since AWS Support takes a little while.

