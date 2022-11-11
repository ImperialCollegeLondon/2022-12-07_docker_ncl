---
title: "Creating More Complex Container Images"
teaching: 30
exercises: 30
questions:
- "How can I make more complex container images? "
objectives:
- "Explain how you can include files within Docker container images when you build them."
- "Explain how you can access files on the Docker host from your Docker containers."
keypoints:
- Docker allows containers to read and write files from the Docker host.
- You can include files from your Docker host into your Docker container images by using the `COPY` instruction in your `Dockerfile`.
---

In order to create and use your own container images, you may need more information than
our previous example. You may want to use files from outside the container, 
that are not included within the container image, either by copying the files 
into the container image, or by making them visible within a running container from their 
existing location on your host system. You may also want to learn a little bit 
about how to install software within a running container or a container image. 
This episode will look at these advanced aspects of running a container or building 
a container image. Note that the examples will get gradually
more and more complex -- most day-to-day use of containers and container images can be accomplished
using the first 1--2 sections on this page.

## Using scripts and files from outside the container

In your shell, change to the `sum` folder in the `docker-intro` folder and look at
the files inside.

~~~
$ cd ~/Desktop/docker-intro/sum
$ ls
~~~
{: .language-bash}

This folder has both a `Dockerfile` and a Python script called `sum.py`. Let's say
we wanted to try running the script using a container based on our recently created `alpine-python`
container image.

> ## Running containers
>
> What command would we use to run Python from the `alpine-python` container?
{: .challenge}

If we try running the container and Python script, what happens?

~~~
$ docker container run alice/alpine-python python3 sum.py
~~~
{: .language-bash}
~~~
python3: can't open file 'sum.py': [Errno 2] No such file or directory
~~~
{: .output}

> ## No such file or directory
>
> What does the error message mean? Why might the Python inside the container
> not be able to find or open our script?
>
{: .challenge}

The problem here is that the container and its filesystem is separate from our
host computer's filesystem. When the container runs, it can't see anything outside
itself, including any of the files on our computer. In order to use Python
(inside the container) and our script (outside the container, on our host computer),
we need to create a link between the directory on our computer and the container.

This link is called a "mount" and is what happens automatically when a USB drive
or other external hard drive gets connected to a computer -- you can see the
contents appear as if they were on your computer.

We can create a mount between our computer and the running container by using an additional
option to `docker container run`. We'll also use the variable `${PWD}` which will substitute
in our current working directory. The option will look like this

`--mount type=bind,source=${PWD},target=/temp`

What this means is: make my current working directory (on the host computer) -- the source --
_visible_ within the container that is about to be started, and inside this container, name the
directory `/temp` -- the target.

> ## Types of mounts
> You will notice that we set the mount `type=bind`, there are other types of mount that
> can be used in Docker (e.g. `volume` and `tmpfs`). We do not cover other types of mounts
> or the differences between these mount types in the course as it is more of an advanced
> topic. You can find more information on the different mount types in
> [the Docker documentation](https://docs.docker.com/storage/).
{: .callout}

Let's try running the command now:
~~~
$ docker container run --mount type=bind,source=${PWD},target=/temp alice/alpine-python python3 sum.py
~~~
{: .language-bash}

But we get the same error!
~~~
python3: can't open file 'sum.py': [Errno 2] No such file or directory
~~~
{: .output}

This final piece is a bit tricky -- we really have to remember to put ourselves
inside the container. Where is the `sum.py` file? It's in the directory that's been
mapped to `/temp` -- so we need to include that in the path to the script. This
command should give us what we need:

~~~
$ docker container run --mount type=bind,source=${PWD},target=/temp alice/alpine-python python3 /temp/sum.py
~~~
{: .language-bash}

Note that if we create any files in the `/temp` directory while the container is
running, these files will appear on our host filesystem in the original directory
and will stay there even when the container stops.

> ## Other Commonly Used Docker Run Flags
>
> Docker run has many other useful flags to alter its function.
> A couple that are commonly used include `-w` and `-u`.
> 
> The `--workdir`/`-w` flag sets the working directory a.k.a. runs the command 
> being executed inside the directory specified.
> For example, the following code would run the `pwd` command in a container
> started from the latest ubuntu image in the `/home/alice` directory and print
> `/home/alice`.  If the directory doesn't exist in the image it will create it.
>
> ~~~
> docker container run -w /home/alice/ ubuntu pwd
> ~~~
>
> The `--user`/`-u` flag lets you specify the username you would like to run the
> container as.  This is helpful if you'd like to write files to a mounted folder
> and not write them as `root` but rather your own user identity and group. 
> A common example of the `-u` flag is `--user $(id -u):$(id -g)` which will
> fetch the current user's ID and group and run the container as that user.
> 
{: .callout}

> ## Exercise: Explore the script
>
> What happens if you use the `docker container run` command above
> and put numbers after the script name?
>
> > ## Solution
> >
> > This script comes from [the Python Wiki](https://wiki.python.org/moin/SimplePrograms)
> > and is set to add all numbers
> > that are passed to it as arguments.
> {: .solution}
{: .challenge}

> ## Exercise: Checking the options
>
> Our Docker command has gotten much longer! Can you go through each piece of
> the Docker command above and explain what it does? How would you characterize
> the key components of a Docker command?
>
> > ## Solution
> >
> > Here's a breakdown of each piece of the command above
> >
> > - `docker container run`: use Docker to run a container
> > - `--mount type=bind,source=${PWD},target=/temp`: connect my current working directory (`${PWD}`) as a folder
> > inside the container called `/temp`
> > - `alice/alpine-python`: name of the container image to use to run the container
> > - `python3 /temp/sum.py`: what commands to run in the container
> >
> > More generally, every Docker command will have the form:
> > `docker [action] [docker options] [docker container image] [command to run inside]`
> >
> {: .solution}
{: .challenge}

> ## Exercise: Interactive jobs
>
> Try using the directory mount option but run the container interactively.
> Can you find the folder that's connected to your host computer? What's inside?
>
> > ## Solution
> >
> > The docker command to run the container interactively is:
> > ~~~
> > $ docker container run --mount type=bind,source=${PWD},target=/temp -it alice/alpine-python sh
> > ~~~
> > {: .language-bash}
> >
> > Once inside, you should be able to navigate to the `/temp` folder
> > and see that's contents are the same as the files on your host computer:
> > ~~~
> > /# cd /temp
> > /# ls
> > ~~~
> > {: .language-bash}
> > ~~~
> > Dockerfile  sum.py
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

Mounting a directory can be very useful when you want to run the software inside your container on many different input files.
In other situations, you may want to save or archive an authoritative version of your data by adding it to the container image permanently.  That's what we will cover next.

## Including your scripts and data within a container image

Our next project will be to add our own files to a container image -- something you
might want to do if you're sharing a finished analysis or just want to have
an archived copy of your entire analysis including the data. Let's assume that we've finished with our `sum.py`
script and want to add it to the container image itself.

In your shell, you should still be in the `sum` folder in the `docker-intro` folder.
~~~
$ pwd
~~~
{: .language-bash}
~~~
/Users/yourname/Desktop/docker-intro/sum
~~~
{: .output}

First, copy the Dockerfile you used for the previous example to this directory:
~~~
$ cp ../basic/Dockerfile .
~~~
{: .language-bash}

Let's add a new line to the `Dockerfile` we've been using so far to create a copy of `sum.py`.
We can do so by using the `COPY` keyword:

~~~
COPY sum.py /home
~~~

This line will cause Docker to copy the file from your computer into the container image's
filesystem. 

Add this line after the final `RUN` line in your Dockerfile. The final Dockerfile should 
look like:

~~~
FROM alpine
RUN apk add --update python3 py3-pip python3-dev
RUN pip install cython
COPY sum.py /home
CMD ["python3", "--version"]
~~~

Let's build the container image like before, but give it a different name:

~~~
$ docker image build -t alice/alpine-sum .
~~~
{: .language-bash}

> ## The Importance of Command Order in a Dockerfile
> 
> When you run `docker build` it executes the build in the order specified
> in the `Dockerfile`.
> This order is important for rebuilding and you typically will want to put your `RUN` 
> commands before your `COPY` commands.
> 
> Docker builds the layers of commands in order.
> This becomes important when you need to rebuild container images.
> If you change layers later in the `Dockerfile` and rebuild the container image, Docker doesn't need to 
> rebuild the earlier layers but will instead used a stored (called "cached") version of
> those layers.
> 
> For example, in an instance where you wanted to copy `multiply.py` into the container 
> image instead of `sum.py`.
> If the `COPY` line came before the `RUN` line, it would need to rebuild the whole image. 
> If the `COPY` line came second then it would use the cached `RUN` layer from the previous
> build and then only rebuild the `COPY` layer.
> 
{: .callout}


> ## Exercise: Did it work?
>
> Can you remember how to run a container interactively? Try that with this one.
> Once inside, try running the Python script.
>
> > ## Solution
> >
> > You can start the container interactively like so:
> > ~~~
> > $ docker container run -it alice/alpine-sum sh
> > ~~~
> > {: .language-bash}
> >
> > You should be able to run the python command inside the container like this:
> > ~~~
> > /# python3 /home/sum.py
> > ~~~
> > {: .language-bash}
> >
> {: .solution}
{: .challenge}

This `COPY` keyword can be used to place your own scripts or own data into a container image
that you want to publish or use as a record. Note that it's not necessarily a good idea
to put your scripts inside the container image if you're constantly changing or editing them.
Then, referencing the scripts from outside the container is a good idea, as we
did in the previous section. You also want to think carefully about size -- if you
run `docker image ls` you'll see the size of each container image all the way on the right of
the screen. The bigger your container image becomes, the harder it will be to easily download.

> ## Security warning
> Login credentials including passwords, tokens, secure access tokens or other secrets
> must never be stored in a container image. If secrets are stored, they are at high risk to
> be found and exploited when made public.
{: .callout}

> ## Copying alternatives
>
> Another trick for getting your own files into a container image is by using the `RUN`
> keyword and downloading the files from the internet. For example, if your code
> is in a GitHub repository, you could include this statement in your Dockerfile
> to download the latest version every time you build the container image:
>
> ~~~
> RUN git clone https://github.com/alice/mycode
> ~~~
>
> Similarly, the `wget` command can be used to download any file publicly available
> on the internet:
>
> ~~~
> RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.10.0/ncbi-blast-2.10.0+-x64-linux.tar.gz
> ~~~
>
> Note that the above `RUN` examples depend on commands (`git` and `wget` respectively) that 
> must be available within your container: Linux distributions such as Alpine may require you to 
> install such commands before using them within `RUN` statements.
{: .callout}

## Docker image compatibility

Although we have noted that one of the strengths of container images is that you can build them and then 
reuse on different systems running Linux, macOS and Windows, there is a particular compatibility issue
that you need to be aware of related to different processor types.

Until quite recently, this was not a big issue as almost all systems used x86_64 architecture processors (Intel and AMD)
but now that many Apple laptops and workstations use Arm64-based architecture processors (branded as *Apple Silicon*) you
may need to be aware of these issues. This applies for the cases where you are generating container images on systems with
an Arm64 architecture - where people you are sharing container images with may be using x86_64 architectures - and when you are generating
container images on systems with an x86_64 architecture - where some of the people using the container images may be 
using Arm64 architecture system.

> ## Docker architecture (platform) labels
>
> Docker supports different processor architectures as part of the *platform* specification of the container image. The platform
> consists of a specification of the operating system kernel (usually `linux`) and the processor architecture. Container images
> for the x86_64 architecture usually use the `linux/amd64` platform and those for the Arm64 architecture usually use the
> `linux/arm64` platform.
{: .callout}

As container images contain compiled software, they are typically only compatible with systems that have the 
same architecture as the architecture specified when building the container image. When you do not
explicitly specify the container image architecture then it will be built using the same architecture
as the system where the container image was built. So, for all the container images you have built so far,
they will be built for the same architecture as your system - typically x86 unless you are using 
an Arm-based Mac.

You typically have two options to build Docker container images to support both x86 and Arm architectures:

- Build multiple copies of the container image, each with a different architecture
- Build a single container image that supports multiple architectures

> ## Running x86 containers on Arm-based Mac 
>
> The situation is slightly more complicated than this as Arm-based Mac *can* successfully run many x86 architecture
> containers as macOS has a piece of software called [Rosetta](https://en.wikipedia.org/wiki/Rosetta_(software))
> that can translate x86 instructions to Arm instructions on-the-fly. However, this does not always work well
> so, if you can generate Arm-specific container images, they will generally have a better success rate on Macs.
{: .callout}

### Build a Docker container image for a particular processor type

This approach is the simplest as it does not require you to use more complex container image build systems
(i.e. `buildx`, described below). The disadvantage is that you need to run multiple build commands and label
your container images differently. When using your container images, other people must be aware of the differences
and make sure they use the container image with the correct format.

You specify the architecture (*platform*) to build for with the `--platform` flag to the `docker image build` command.
For example, to build an x86 architecture version of the `alpine-sum` container image, we would use:

```
docker image build --platform linux/amd64 -t alice/alpine-sum-x86 .
```
{: .language-bash}

You can check that the image has been built for the correct architecture using the `docker image inspect`
command. Look for the line containing "Architecture":

~~~
docker image inspect alice/alpine-sum-x86
~~~
{: .language-bash}
~~~
...trim...
        "Architecture": "amd64",
        "Os": "linux",
...trim...
~~~
{: .output}

> ## Exercise: Build the Arm version of the `alpine-sum` container image
>
> Build a version of the `alpine-sum` container image for the arm64 platform. Can
> you check that the image has been successfully built with the correct architecture?
>
> > ## Solution
> >
> > The correct option is `--platform linux/arm64`:
> > ~~~
> > $ docker image build --platform linux/arm64 -t alice/alpine-sum-arm64 .
> > ~~~
> > {: .language-bash}
> >
> > You can check the image has the correct architecture with:
> > ~~~
> >docker image inspect alice/alpine-sum-arm64
> > ~~~
> > {: .language-bash}
> > ~~~
> > ...trim...
> >         "Architecture": "arm64",
> >         "Os": "linux",
> > ...trim...
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}

### Build a multi-format Docker container image

Most of the container images on Docker Hub from major providers (e.g. the official Python container images)
typically provide a single multi-platform container image. When you pull the image from Docker Hub, your
Docker instance knows to get the version of the multi-platform container image that best matches your local
platform. (On an Arm architecture Mac, if an Arm version of the container image does not exist it will pull
the `amd64` version and try to run it using the Rosetta software.)

In order to build multi-platform container images we need to use a different builder tool as the default 
Docker container image build tool does not support building multi-platform images. If you are using a recent
version of Docker Desktop on macOS or Windows this build tool comes pre-installed. However, you need to 
initialise a new builder that uses this tool. We do this with the `docker buildx create` command:

~~~
docker buildx create --name multiplatform 
~~~
{: .language-bash}
~~~
multiplatform
~~~
{: .output}

Next, you need to tell Docker to use the new builder engine with:

~~~
docker buildx use multiplatform
~~~
{: .language-bash}

One caveat when building multi-platform container images: they cannot be stored locally so they must be
pushed to Docker Hub (or another online repository) as part of the build process.

Now, finally, we are ready to build our multi-platform container image using our new builder engine and
push it to Docker Hub:

~~~
docker buildx build --platform linux/amd64,linux/arm64 -t aturnerepcc/alpine-sum-multi --push .
~~~
{: .language-bash}

Note:

- We specify the two platforms we want to build for: `--platform linux/amd64,linux/arm64`
- We tell Docker to push to Docker Hub once the container images are built: `--push`

Once this has been done, you can inspect the container image on Docker Hub and verify
it contains variants for both platforms. If you go to Docker Hub and select the container
image. Use `See all` in the "Tags and scans" section to reveal what platforms are supported
for the container image.

If you then pull the container image, you will get the version that best matches your
current platform. For example, on my M1 Mac, I get the `arm64` version:


~~~
docker pull aturnerepcc/alpine-sum-multi
docker image inspect aturnerepcc/alpine-sum-multi
~~~
{: .language-bash}
~~~
Using default tag: latest
latest: Pulling from aturnerepcc/alpine-sum-multi
9b18e9b68314: Already exists 
41fd814ce19c: Pull complete 
cb7960f11e5f: Pull complete 
30433674418c: Pull complete 
Digest: sha256:d7110318ffacf1dae113d8569bd0e02056ded923ca481957423d01a65302f727
Status: Downloaded newer image for aturnerepcc/alpine-sum-multi:latest
docker.io/aturnerepcc/alpine-sum-multi:latest
...trim...
        "Architecture": "arm64",
        "Os": "linux",
...trim...
~~~
{: .output}


> ## Exercise: Build a multi-platform container image
>
> Build a multi-platform version of one of the container images we have built so far in this
> lesson and push it to Docker Hub. Can you check on Docker Hub and see that the container is
> genuinely multi-platform? If another course attendee has a laptop with a different architecture
> to yours, do they get the correct version of the container image if they pull from Docker Hub?
>
{: .challenge}

### Important: restoring the standard Docker builder

To return to the standard Docker builder - you probably want to do this!

~~~
docker buildx use default
~~~
{: .language-bash}

Next time you want to build a multi-platform container image you just need to switch back to 
the alternative builder with:

~~~
docker buildx use multiplatform
~~~
{: .language-bash}

You do not need to initialise it again.


## More fancy `Dockerfile` options

We can expand on the example above to make our container image even more "automatic".
Here are some ideas:

### Make the `sum.py` script run automatically

~~~
FROM alpine
RUN apk add --update python3 py3-pip python3-dev
COPY sum.py /home

# Run the sum.py script as the default command
CMD ["python3", "/home/sum.py"]
~~~

Build and test it:
~~~
$ docker image build -t alpine-sum:v1 .
$ docker container run alpine-sum:v1
~~~
{: .language-bash}

You'll notice that you can run the container without arguments just fine,
resulting in `sum = 0`, but this is boring. Supplying arguments however
doesn't work:
~~~
docker container run alpine-sum:v1 10 11 12
~~~
{: .language-bash}
results in
~~~
docker: Error response from daemon: OCI runtime create failed:
container_linux.go:349: starting container process caused "exec:
\"10\": executable file not found in $PATH": unknown.
~~~
{: .output}

This is because the arguments `10 11 12` are interpreted as a
*command* that replaces the default command given by `CMD
["python3", "/home/sum.py"]` in the image.

To achieve the goal of having a command that *always* runs when a
container is run from the container image *and* can be passed the arguments given on the
command line, use the keyword `ENTRYPOINT` in the `Dockerfile`.

~~~
FROM alpine

COPY sum.py /home
RUN apk add --update python3 py3-pip python3-dev

# Run the sum.py script as the default command and
# allow people to enter arguments for it
ENTRYPOINT ["python3", "/home/sum.py"]

# Give default arguments, in case none are supplied on
# the command-line
CMD ["10", "11"]
~~~

Build and test it:
~~~
$ docker image build -t alpine-sum:v2 .
# Most of the time you are interested in the sum of 10 and 11:
$ docker container run alpine-sum:v2
# Sometimes you have more challenging calculations to do:
$ docker container run alpine-sum:v2 12 13 14
~~~
{: .language-bash}

> ## Overriding the ENTRYPOINT
> Sometimes you don't want to run the
> image's `ENTRYPOINT`. For example if you have a specialized container image
> that does only sums, but you need an interactive shell to examine
> the container:
> ~~~
> $ docker container run -it alpine-sum:v2 /bin/sh
> ~~~
> {: .language-bash}
> will yield
> ~~~
> Please supply integer arguments
> ~~~
> {: .output}
> You need to override the `ENTRYPOINT` statement in the container image like so:
> ~~~
> $ docker container run -it --entrypoint /bin/sh alpine-sum:v2
> ~~~
> {: .language-bash}
{: .callout}

### Add the `sum.py` script to the `PATH` so you can run it directly:

~~~
FROM alpine

RUN apk add --update python3 py3-pip python3-dev

COPY sum.py /home
# set script permissions
RUN chmod +x /home/sum.py
# add /home folder to the PATH
ENV PATH /home:$PATH
~~~

Build and test it:
~~~
$ docker image build -t alpine-sum:v3 .
$ docker container run alpine-sum:v3 sum.py 1 2 3 4
~~~
{: .language-bash}

> ## Best practices for writing Dockerfiles
> Take a look at Nüst et al.'s "[_Ten simple rules for writing Dockerfiles for reproducible data science_](https://doi.org/10.1371/journal.pcbi.1008316)" \[1\] 
> for some great examples of best practices to use when writing Dockerfiles. 
> The [GitHub repository](https://github.com/nuest/ten-simple-rules-dockerfiles) associated with the paper also has a set of [example `Dockerfile`s](https://github.com/nuest/ten-simple-rules-dockerfiles/tree/master/examples) 
> demonstrating how the rules highlighted by the paper can be applied.
>
> <small>[1] Nüst D, Sochat V, Marwick B, Eglen SJ, Head T, et al. (2020) Ten simple rules for writing Dockerfiles for reproducible data science. PLOS Computational Biology 16(11): e1008316. https://doi.org/10.1371/journal.pcbi.1008316</small>
{: .callout}

{% include links.md %}
