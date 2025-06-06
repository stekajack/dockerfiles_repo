
The images built using these Dockerfiles are available on DockerHub under my user: https://hub.docker.com/u/stekajack

To pull a docker image from the terminal you can run something like:
```bash
docker pull user/image_name:tag
```

To build an image locally based on a Dockerfile, run this command from the same location where your Dockerfile is (or provide a path to it instead of the '.'):
```bash
docker build --no-cache -t user/image_name:tag .
```

The `no-cache` flag is not always necessary, but keep in mind Docker has no way of knowing if you made a push or a pull request was merged to a repository you're cloning with the Dockerfile.  
The `no-cache` flag ensures the image is built from scratch. Images rebuilt from cache will get bigger over time because of what I can only describe as random leftover crap.

Things to keep in mind, that are not so obvious but can save you time:

Docker supports users as a concept, but is not really well suited for that. My strong recommendation is to keep your image build with root privileges.  
AI will tell you this is a security risk, but AI is stupid. It's a security risk only as far as the machine you're running on is compromised. Otherwise, trust yourself and live a happy life.

In Dockerfiles, you'll see the following two flags set:
```dockerfile
ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1
```

If you try to run without them your MPI processes will fail because running MPI processes as root is considered unsafe (can in principle crash your system irreversibly). This is a sensible warning, and no sysadmin would ever let you run as root on a cluster.  
However, from within a container, the only thing you can break is the container itself. There's absolutely no potential to mess up your machine. On a cluster, you'll most likely be running your images in Apptainer/Singularity, where this is not an issue (more detail later).

At the end you'll see:
```dockerfile
ENTRYPOINT ["/bin/bash", "-c", "tail -f /dev/null"]
```
This command is unnecessary and should be removed for CI! For runtime, however, it's convenient because it keeps the container running, even if you don't execute a specific command. So your container can be used as a fully isolated runtime!

Helpful docker commands you might want to use:
```bash
docker images # see images on your local machine
docker run -it user/image_name:tag # run local container from local image
docker run -v path_to_folder_host:path_to_folder_container --gpus all user/image_name:tag # bind a folder in you container to a folder on the host and enable GPU support
docker tag image_name user/image_name:tag # tag an image to a particular user/tag (kinda like creating a repository on GitHub before pushing to it)
docker push user/image_name:tag # push to your DockerHub repo
```

Now onto using your docker images in HPC environments:

Typically clusters have Apptainer/Singularity or SingularityPro configured and ready to use.  
- Vienna Scientific Cluster (wiki: https://docs.vsc.ac.at) has several versions of Apptainer configured (available as a module).  
- Leonardo (wiki: https://wiki.u-gov.it/confluence/display/SCAIUS/LEONARDO+User+Guide) has SingularityPro (always available, no loading necessary).

This is purely a nominal difference; the API is almost identical. I use Apptainer for the rest of the FAQ, but you can assume all commands work the same. You only need to change the first invocation word (Apptainer -> Singularity).

To pull a docker image from DockerHub:
```bash
apptainer pull image_name.sif docker://user/image_name:tag
```

This converts the image into the `.sif` format, used by Apptainer. The conversion itself is quite safe. If you're doubtful, it's not a lot of work to make the same image directly with Apptainer. AI can translate Dockerfiles confidently.

To start a container on a login node, for example:
```bash
apptainer instance start image_name.sif container_name
```
Conversely, to stop it:
```bash
apptainer instance stop container_name
```
You can have any number of independently running containers. To see them all:
```bash
apptainer instance list
```

Strongly recommended: explicitly shut down running containers before you logout from the node, otherwise, they can persist and run in the background for quite a while!

To attach a shell to your running container ("entering" the container):
```bash
apptainer shell instance://container_name
```

When you attach a shell, you'll notice that everything inside looks exactly like your account on the login node. That's because Apptainer is designed to mimic the user running the image and loads the complete environment from which it is run.

Only the `$HOME` is mounted automatically! Data partitions (`$DATA`, `$WORK`) are NOT mounted automatically! You need to mount these partitions manually.  
Suggestion: create an obvious folder in your `$HOME` (e.g., `DATA_VIEW`) and explicitly bind your data partition:
```bash
apptainer instance start --bind $DATA:$HOME/DATA_VIEW image_name.sif container_name
```

Example of executing a command without first attaching a shell:
```bash
apptainer exec image_name.sif /proc/self/root/home/user/espresso/build/pypresso hello.py
```

Example SLURM submission file for Apptainer:
```bash
#!/bin/bash
#SBATCH -J apptainer_job
#SBATCH -N 1
#SBATCH --ntasks-per-node=#
#SBATCH --ntasks-per-core=#
#SBATCH --partition=partition_name
#SBATCH --qos=qos_value

module purge
module load apptainer/1.1.9-gcc-12.2.0-ab3ze5o

apptainer exec --bind $DATA:$HOME/DATA_VIEW $HOME/image_name.sif bash -c 'mpirun -np # $ESPRESSO_PATH/pypresso $HOME/hello.py &> $HOME/DATA_VIEW/output.txt &'

wait
```

When running MPI jobs, the following flags might help:
- `--bind-to none`: Uses the container OS scheduler (usually Ubuntu’s CFS scheduler).
- `--report-bindings`: Prints how each rank sees the underlying sockets/CPUs.

Happy Dockerizing and Containerizing!
