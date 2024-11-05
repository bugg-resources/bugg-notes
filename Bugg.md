* Drawing of the M12 cable
* Placement of the Antenna


* Laser engraving


* Test, first article PCBA hybrid test approach - Give Griffin an procedure for basic bring-up - flash CM4, check it boots, check USB uart,


# Firmware
We need to be able to build firmware images in a sane, fast, reproducible manner. Our current approach is to install RPi OS onto an actual Pi, hack the config around until it's as desired, then back up the filesystem for flashing onto other Pi's.

Clearly this isn't the best arrangement.

## pi-gen
This is the tooling that the RPi org use to make their own firmware images. It's designed to permit the creation of custom images without too much pain.
https://github.com/RPi-Distro/pi-gen

pi-gen only works on Debian or Ubuntu (which is Debian-based). You can run it in Docker if you want to use a different platform.

pi-gen doesn't build the kernel - it includes a pre-built kernel image into the build. It uses `chroot`to create an isolated folder which behaves like the OS `/` folder for one process then, in that process, it uses APT to install a load of packages into it. Creating this initial folder is called "bootstrap".

pi-gen goes through several stages, 0-6. After step 2, you get the "Raspberry Pi OS Lite" image, which is CLI-only, but has all of the device drivers, working networking, etc. This is what we want as a starting point. 

After stage 3 there is a basic desktop, and 4 and 5 develop this to add a full DE, office software, etc.

## Github Actions
We could spin up a Debian environment in Docker or a VM, but it makes even more sense to run pi-gen in a Github Action. This is a CI/CD tool which can operate on codebases to do just about anything imaginable.

Following the documentation
https://docs.github.com/en/actions/learn-github-actions
and this example of a custom RPi build
https://github.com/hyperion-project/HyperBian

Github Actions provide a "Runner", which executes the automation that you describe with one/more YAML files. You can define "jobs", which run in parallel, each composed of "steps", which run sequentially. All rather nice.

I've managed to create a Github Action that successfully builds RPi OS with some very slight modifications.

### ssh access for debugging the Runner
For some reason, this doesn't seem like a very common workflow, and certainly not something provided by Github, but I can't really imagine an alternative to shelling into the Github Action VM (called the Runner) and dicking about to find out why your CI scripts are failing.

https://github.com/mxschmitt/action-tmate

The concept of the runner is that it comes to life as a blank slate, gets conditioned for use, gets used, then dies; all within a short space of time. Github sell a certain amount of time on these runners; e.g., the free tier gives you 2000mins/month. Maybe they don't want you shelling in and leaving the terminal open for hours eating up time. I dunno, it's weird.

### apt-cacher
Github has a caching system which I'm struggling to understand. It currently takes about 40 minutes to build our image. Most of the time is spent by APT fetching and installing packages into the image. `pi-gen` supports a connection to `apt-cacher`, which sits on the network providing locally cached versions of packages to stop `apt` needing to fetch them from the internet.

I don't know how much time this will actually save, because, I believe, the Github Action VM has a lot more network than disk/CPU, but maybe it's worth a try.

Ok, I've installed apt-cacher in the runner. It caches `.deb` files into `/var/cache/apt`. Now let's see if I can get the firmware image build to cache...

So far, no luck.
### Google Actions cache

No, scratch that, we don't need apt-cacher. pi-gen supports apt-cacher because it is designed to run on an ordinary system. GA, on the other hand, can directly cache some 

Whilst building the image, packages are being archived here:
`/home/runner/work/bugg-os/bugg-os/pi-gen/work/BuggOS/stage0/rootfs/var/cache/apt/archives`

This happens normally, without apt-cacher. I suspect, however, that pi-gen may delete this folder between runs... Maybe, maybe not. What's the easiest way to find out?

Next we need to tell Github Actions to cache that folder (as well as the ones for the other stages 1, 2, etc.)

Ok I haven't got apt-cacher to work, but we don't actually need it. GA can just directly cache files on the filesystem. Normally, when the runner starts, it is a virgin container with a clean filesystem. GA caching allows it to boot up with some files preserved from the previous run. We can make use of this to retain the apt cache between runs.

### Does pi-gen delete its apt cache before/after a run?
If it does, we have no option but to use apt-cacher (other than just running the build on a faster machine).

So how to answer this question? We need to run the build on an actual machine, rather than an ephermeral runner, then look at what happens to the files .

First I tried to do this with docker. I hit this showstopper:
```
root@c107956f3302:~/bugg-os/pi-gen# ./build.sh 
Module binfmt_misc not loaded in host
Please run:
  sudo modprobe binfmt_misc
modprobe binfmt_misc
modprobe: FATAL: Module binfmt_misc not found in directory /lib/modules/6.1.13-200.fc37.x86_64
```
  
I decided to stop fucking around with docker and try an AWS instance instead. I've spun up the beefiest one available (at least available before you have to chat to someone at Amazon). It's not _that_ impressive - I think it's something like quad core 32GB.

Ok, I've just built the code in an AWS t3.2xlarge instance - 8 vCPU's, 32GB.
Interestingly, it took about the same time (34m21s) to build the image - about the same as the Github free runner. So clearly even the cheapest Github runner is fairly decent. 

```
root@ip-172-31-29-240:/home/ubuntu/bugg-os/pi-gen# ls work/BuggOS/stage0/rootfs/var/cache/apt/archives/ |wc -l
299
```

Let's see what happens to the apt cache when we restart the build:

Ok that all fucked up - the instance ran out of disk. I increased it, but in the end it was just easier to spin up a new one. Now running a c5a.8xlarge - 32 vCPU's, 64GB.

Meh - same time on much more powerful system. Clearly this thing is not engineered to use multiple CPU's.

Let's run a second time without changing anything. Good news - it doesn't seem to have erased the cache. I discovered there's a CLEAN flag, and it seems that if it's not set, the files aren't deleted. So the second run of this build should be quicker. Let's see...
Second run takes 11:55. That's tolerable. 

So, let's get Google Actions to cache whatever persists between runs. 

Looking at what CLEAN does in build.sh, it removes:
```
.pc
./*-pc
${ROOTFS_DIR}
ROOTFS_DIR="${STAGE_WORK_DIR}"/rootfs
STAGE_WORK_DIR="${WORK_DIR}/${STAGE}"
```
All of that stuff is under /work, so let's just try to cache that whole thing. It's 6GB.
# Other options
Docker something something
AWS CodeBuild
Github Actions with a bigger, paid-for runner
"Reusable" GA Workflows
