# general-kronos
General tips and practical things for working on kronos

Date: April 2020

Last updated: 18/06/2020

Authors: Manuela Tan


## General description and purpose
This is a general description of handy steps when using the kronos server at UCL.


# 1. Connecting to kronos

If you are just starting, you will need to ask Hallgeir Jonvik to set you up a username and password on kronos, and to get the IP address (here just used [kronos_IP_address] - replace this with the actual IP address).

## From a Mac

Connect to the UCL VPN using Cisco AnyConnect. Open a terminal window. Then ssh to the server address with your own username. You will be prompted to type in your password (this will be masked)
```
ssh username@kronos_IP_address
```

Use Cmd + K and this address smb://[kronos_IP_address]/kronos to connect to the drive through the Finder.


## From a Windows PC

Install Putty. Connect to the UCL VPN.

Open Putty and put in the server address in the Host Name box, connection type SSH. You can save your session details by clicking the Save option (name it kronos or similar) then double click this or select Load from the Saved Sessions box next time.

Click Connect. This will open a terminal window and you will be prompted to put in your username and password (this will be masked).

To see the Kronos folders in your directory, go to My Computer and 'Map Network Drive'. In the Folder box, put the kronos address \\\kronos_IP_address.


## Change to your working directory

When you first log into kronos, you are in your home directory but this is not on the kronos directory.

Change directory using cd to your own folder on kronos.
```
cd /data/kronos/mtan/
```


# 2. Linking to the UCL Research Data Store (RDS)
Use this to free up space on Kronos. Raw data should be stored on the RDS not on Kronos.
Ask Hallgeir Jonvik if you have any issues with this.

## Mount the RDS onto kronos

First make a directory on kronos that you want to mount the RDS onto. You can call this whatever you want.

```
mkdir /data/kronos/mtan/mount_rd00pi_3
```

Then link the RDS drive to this folder on kronos. Put in your UCL username (skxxxx) 

```
sudo mount -t cifs -o user=skxxxx,noperm,domain=ad.ucl.ac.uk //ssh.rd.ucl.ac.uk/ritd-ag-project-projectname /data/kronos/mtan/mount_rd00pi_3/
```
You will be prompted to put in your password - this is your UCL password not kronos password.

You will then be able to access the RDS folders from kronos. You only need to do this once.

## Copy files from the RDS onto kronos

Once you have mounted the RDS onto kronos in your directory, copying files from RDS to kronos is the same as copying files within kronos.

This is much quicker than copying files using regular copy.

```
cp /data/kronos/mtan/mount_rd00pi_3/NeuroChip_rawdata/file1.txt 
cp /data/kronos/mtan/mount_rd00pi_3/NeuroChip_rawdata/file2.txt
cp /data/kronos/mtan/mount_rd00pi_3/NeuroChip_rawdata/file3.txt 
/data/kronos/mtan/newlocation
```

## Copy files from kronos to the RDS

To copy files from kronos onto the RDS, make sure you have mounted the RDS onto kronos in your folder.

First make a directory in the mounted drive where you want to copy files into.

```
mkdir /data/kronos/mtan/mount_rd00pi_3/DESTINATION
```

Use rsync to copy files from kronos to the RDS mounted drive. Remove --dry-run once you know it works as expected.

```
rsync -vahP /data/kronos/mtan/SOURCE/ /data/kronos/mtan/mount_rd00pi_3/DESTINATION/ --dry-run
```

# 3. Copying from local PCs to kronos and vice versa

This works on macs, I don't know how to do it on Windows. 

In your regular terminal window:

To copy from kronos to laptop
```
scp username@kronos_IP_address:/data/kronos/mtan/FILENAME /Users/manuela/Desktop/
```

To copy from laptop to kronos
```
scp /Users/manuela/LOCATION/FILENAME username@kronos_IP_address:/data/kronos/mtan/DESTINATION
``` 

# 4. Using nohup to keep running jobs after you disconnect from the server

nohup (stands for 'no hang up') is a command that lets you continue running a command or script even after you disconnect from the server. This can be really useful for commands/jobs that take a long time (e.g. working with large datasets). The job will continue running even after you disconnect from kronos, turn off your computer etc.

https://linux.101hacks.com/unix/nohup-command/

The general format of your code should be:
```
nohup command &
```

Any output (e.g. stuff that is usually printed in your terminal, like plink logs) or error messages will be saved to a file called nohup.out

You can use nohup with a regular command in the terminal, e.g.
```
nohup plink --bfile FILENAME \
--maf 0.01 \
--make-bed \
--out NEWFILENAME &
```

You can also use nohup with a script, e.g.
```
nohup sh myshellscript.sh &
```

or
```
nohup Rscript myrscript.r &
```

You can change the name of the output file (e.g. if you are running multiple commands/jobs with nohup).
```
nohup sh myshellscript.sh &> aDifferentName.out&
```

# 5. Using the job scheduler on kronos

If you want to run many jobs/processes in parallel, rather than in the terminal, you can use the cluster. Kronos uses Sun Grid Engine to schedule and manage jobs and queues.

http://bioinformatics.mdc-berlin.de/intro2UnixandSGE/sun_grid_engine_for_beginners/README.html

Basically, if you have many jobs that might take a long time and they are independent of each other, you can use this. E.g. if you want to run a big GWAS but on subsets of the genome (e.g. 50k SNPs at a time).

First you need to set up the 'son of grid engine' path in your home directory, e.g. /data/home/mtan. This will make a .profile file.

```
cat ~/.profile
export SGE_ROOT=/opt/sge
export PATH=$PATH:$SGE_ROOT/bin/lx-amd64
ulimit -u 64000
```
You then need to log in with a new session before it works. You only need to do this once.


To submit a job, the general format is:
```
qsub scriptname.sh
```

You specify the resources you need beforehand
```
qsub -pe make 2 -cwd GWASscript_0.r
```
* -pe make 2 makes 2 cores available for your script

* -cwd means will write the output to the current directory.

You can also specify resources for memory and other things. Anything after the script name in qsub is taken as an argument, and these should be defined in your script, e.g. using args in R.

Recommend to use the full path names for your files in your scripts because sometimes qsub gets confused about your working directory.


Once you submit a job, it will print output to two files: scriptname.e and scriptname.o. These are the standard error and standard output (not indicative of actual error and output). You can head or tail these to see any output. 

Use qstat to check the status of jobs
```
qstat
```

The output will show you the job number; r means it is running and qw means it is queued.


Your script should start with this as the first line - this is to tell the program what shell to use
```
#/bin/bash
```

If your script is an R script, use
```
#/bin/Rscript
```

Ask David Murphy if you have any questions about using the cluster on kronos.
