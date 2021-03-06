# qbatch
Execute shell command lines in parallel on SGE/PBS clusters

[![Travis CI build status](https://travis-ci.org/pipitone/qbatch.svg?branch=master)](https://travis-ci.org/pipitone/qbatch)

qbatch is a tool for executing commands in parallel across a compute cluster.
It takes as input a list of *commands* (shell command lines or executable
scripts) in a file or piped to qbatch. The list of commands are divided into
arbitrarily sized *chunks* which are submitted as jobs to the cluster either as
individual submissions or an array. Each job runs the commands in its chunk in
parallel. Commands can also be run locally on systems with no cluster
capability.

## Installation

```sh 
$ pip install qbatch
```

## Dependencies
qbatch requires python (2.7 or 3) and [GNU Parallel](https://gnu.org/s/parallel).
For Torque/PBS and gridengine clusters, qbatch requires job submission access
via the ``qsub`` command. For Slurm workload manager, qbatch depends on the ``sbatch`` command.

## Environment variable defaults
qbatch supports several environment variables to customize defaults for your
local system.

```sh
$ export QBATCH_PPJ=12                   # requested processors per job
$ export QBATCH_CHUNKSIZE=$QBATCH_PPJ    # commands to run per job
$ export QBATCH_CORES=$QBATCH_PPJ        # commonds to run in parallel per job
$ export QBATCH_MEM="0"                  # requested memory per job
$ export QBATCH_MEMVARS="mem"            # memory request variable to set
$ export QBATCH_SYSTEM="pbs"             # queuing system to use ("pbs", "sge", or "slurm")
$ export QBATCH_NODES=1                  # (PBS-only) nodes to request per job
$ export QBATCH_PE="smp"                 # (SGE-only) parallel environment name
$ export QBATCH_QUEUE="1day"             # (Slurm-only) cluster-specific name of queue (--qos option)
```

These correspond to the same named options in the qbatch help output above.

## Some examples:
```sh
# Submit an array job from a list of commands (one per line)
# Generates a job script in ./.scripts/ and job logs appear in ./logs/
$ qbatch commands.txt

# Set the walltime for each job
$ qbatch -w 3:00:00 commands.txt

# Run 24 commands per job
$ qbatch -c24 commands.txt

# Request 1# Run 24 commands per job, but run 12 in parallel at a time
$ qbatch -c24 -j12 commands.txt

# Start jobs after successful completion of existing jobs with names starting with "stage1_"
$ qbatch --afterok 'stage1_*' commands.txt

# Pipe a list of commands to qbatch 
$ parallel echo process.sh {} ::: *.dat | qbatch -

# Run jobs locally with GNU Parallel, 12 commands in parallel
$ qbatch -b local -j12 commands.txt

# Many options don't make sense locally: chunking, individual vs array, nodes,
# ppj, highmem, and afterok are ignored
```
