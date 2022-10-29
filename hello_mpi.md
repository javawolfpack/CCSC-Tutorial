# MPI Hello World

Now that you have a small cluster let's test that it works.

## Navigate to our shared directory path

MPI relies on the same path for files on all nodes, the easiest way to do this is a shared folder between all the nodes, which for this tutorial was configured as */mnt/mpi_shared*.

```bash
bryandixon@ccsc1:~/CCSC-Tutorial$ cd /mnt/mpi_shared
bryandixon@ccsc1:/mnt/mpi_shared$
```

## Get Hello World Code

Now let's grab a hello world mpi program from an [MPI tutorial](https://github.com/mpitutorial/mpitutorial/blob/gh-pages/tutorials/mpi-hello-world/code/mpi_hello_world.c)

```bash
bryandixon@ccsc1:/mnt/mpi_shared$ wget https://raw.githubusercontent.com/mpitutorial/mpitutorial/gh-pages/tutorials/mpi-hello-world/code/mpi_hello_world.c
```

Now we have the MPI code.

## Compile and Run MPI code

Now let's compile our MPI code:

```bash
bryandixon@ccsc1:/mnt/mpi_shared$ mpicc -o mpi_hello_world mpi_hello_world.c
bryandixon@ccsc1:/mnt/mpi_shared$ ls
mpi_hello_world  mpi_hello_world.c
```

Now we have a *mpi_hello_world* executable. To be able to run the MPI program we need to be able to SSH to all our nodes without password or other prompts. We have configured hostnames for our 3 nodes of *gcp1*, *gcp2*, and *gcp3* so ssh into all 3 of these nodes to make sure there are no prompts any more first.

Now create a hosts file for MPI:

```bash
bryandixon@ccsc1:/mnt/mpi_shared$ printf "gcp1\ngcp2\ngcp3\n" > hosts
bryandixon@ccsc1:/mnt/mpi_shared$ cat hosts
gcp1
gcp2
gcp3
```

This file tells mpi which nodes to run on. Now we can run the code:

```bash
bryandixon@ccsc1:/mnt/mpi_shared$ mpirun -n 3 --hostfile hosts ./mpi_hello_world
Hello world from processor ccsc1, rank 0 out of 3 processors
Hello world from processor ccsc2, rank 1 out of 3 processors
Hello world from processor ccsc3, rank 2 out of 3 processors
```

We now successfully have a cluster and are able to run MPI code on it.
