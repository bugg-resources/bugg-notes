I've been working on Google Actions, and I've made some good progress getting the firmware to build.

This repo contains a GA workflow that uses pi-gen to build a custom Raspberry Pi OS image
https://github.com/jeffmakes/bugg-os

pi-gen is the tool that the Raspberry Pi OS maintainers use to build it.

It's annoyingly slow to run (about 40 min). This is, at least partially, because each time the workflow runs, it is running in a totally fresh environment, with no caching of files or packages. I've tried, and largely failed, to get caching to work properly. Another annoyance is that the GA runner is a bit annoying to work with, during debug, because is not really intended to be used as a general machine (because it really is intended just to exist to run the workflow, Mr Meeseeks style). SSH access is only possible via a hack, and disconnects after a while.

To see if I can make some faster progress, I'll work in an AWS instance for now (which, of course, has a persistent filesystem and sensible SSH access).

# pi-gen references
The docs are fairly decent, but you can find some more hints from other projects:
https://medium.com/@deltazero/making-kioskpi-custom-raspberry-pi-os-image-using-pi-gen-99aac2cd8cb6
https://github.com/hyperion-project/HyperBian

# The AWS instance
It's an ubuntu-jammy-22.04-amd64-server-20230919 on a c5a.8xlarge instance. (32 vCPUs, 64 GB, 10 Gigabit network, USD 1.312/hour).

A 30 GB elastic volume [i-0a7f5cbe336b8e7a9 (ubuntu-2)](https://eu-north-1.console.aws.amazon.com/ec2/home?region=eu-north-1#Instances:instanceId=i-0a7f5cbe336b8e7a9): /dev/sda1 is attached.

## SSH access
A key file is required for SSH access:
`ssh -i "~/tools/soft/amazon-aws/aws-keys.pem" ubuntu@ec2-51-20-5-11.eu-north-1.compute.amazonaws.com `

## Testing with `act`
`act` allows you to run a GA workflow locally. It uses Docker containers to host the runners for each action.

### Installing act
I've installed act into the AWS instance (I had to use the hacky Bash method - the Snap seemed to be broken in some way (segfaults!)).

`curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`

It requires Docker Engine. Follow this installation process for APT:
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
That won't quite work as it's written for Debian. After step 1, edit 
`sudo nvim  /etc/apt/sources.list.d/docker.list` and replace "debian" with "ubuntu". Then apt-get update, and continue with steps 2 and 3.

To run docker containers as non-root:
``` 
sudo groupadd docker
sudo usermod -aG docker $USER
```

### Using act
Listing all actions:
```
ubuntu@ip-172-31-38-230:~/bugg-os$ act -l
Stage  Job ID         Job name       Workflow name          Workflow file  Events           
0      image-builder  image-builder  Bugg OS Release Build  release.yml    workflow_dispatch
```
Running an action. cd into the repo, then:
```
act workflow_dispatch
 ```
 That works, for a bit, then crashes at:
 ```
 [Bugg OS Release Build/image-builder] ⭐ Run Main Checkout pi-gen
[Bugg OS Release Build/image-builder]   🐳  docker cp src=/home/ubuntu/.cache/act/actions-checkout@v4/ dst=/var/run/act/actions/actions-checkout@v4/
[Bugg OS Release Build/image-builder]   🐳  docker exec cmd=[node /var/run/act/actions/actions-checkout@v4/dist/index.js] user= workdir=
[Bugg OS Release Build/image-builder]   💬  ::debug::GITHUB_WORKSPACE = '/home/ubuntu/bugg-os'
[Bugg OS Release Build/image-builder]   💬  ::debug::qualified repository = 'RPi-Distro/pi-gen'
[Bugg OS Release Build/image-builder]   💬  ::debug::ref = 'arm64'
[Bugg OS Release Build/image-builder]   💬  ::debug::commit = 'undefined'
[Bugg OS Release Build/image-builder]   💬  ::debug::clean = true
[Bugg OS Release Build/image-builder]   💬  ::debug::filter = undefined
[Bugg OS Release Build/image-builder]   💬  ::debug::fetch depth = 1
[Bugg OS Release Build/image-builder]   💬  ::debug::fetch tags = false
[Bugg OS Release Build/image-builder]   💬  ::debug::show progress = true
[Bugg OS Release Build/image-builder]   💬  ::debug::lfs = false
[Bugg OS Release Build/image-builder]   💬  ::debug::submodules = false
[Bugg OS Release Build/image-builder]   💬  ::debug::recursive submodules = false
[Bugg OS Release Build/image-builder]   ❗  ::error::Input required and not supplied: token
[Bugg OS Release Build/image-builder]   ❌  Failure - Main Checkout pi-gen
```
This has something to do with github secrets:
https://github.com/actions/checkout/issues/298

I can't be bothered to deal with this right now - I think it'll be more useful to run pi-gen manually for a bit, then revisit the automation later.

### Manually running pi-gen build.sh

I manually followed the verious clone, cp, touch, etc steps from the GA release.yml workflow, to get the local file tree into the same state as it would be in the GA runner.

Let's run pi-gen/build.sh for the first time:
Run 1: real	51m3.025s
Run 2: real	22m4.091s
Run 3: real	22m10.084s
Run 4: real	22m25.541s
export CLEAN=0
real	22m17.906s

Oops, I've been building 32 bit images. Restarting with 64 bit:
Profiling with both time and shellprof:

Run 1:
```
Timings for printed lines:
121.27s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
120.42s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
120.28s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
120.22s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
86.37s: W: Possible missing firmware /lib/firmware/rtl_nic/rtl8153a-2.fw for built-in driver r8152
60.54s: W: Possible missing firmware /lib/firmware/rtl_nic/rtl8153a-2.fw for built-in driver r8152
60.40s: W: Possible missing firmware /lib/firmware/rtl_nic/rtl8153a-2.fw for built-in driver r8152
36.19s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
36.17s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
36.16s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8

real	31m57.993s
user	0m0.026s
sys	0m0.155s
```

Run 2:
```
Timings for printed lines:
120.38s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
120.26s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
120.18s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
119.92s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
16.11s: [22:20:19] Begin /home/ubuntu/bugg-os/pi-gen/export-image/prerun.sh
15.55s: Hit:4 http://archive.raspberrypi.com/debian bookworm InRelease
15.44s: mount: /dev/loop10p1 mounted on /home/ubuntu/bugg-os/pi-gen/work/BuggOS/export-image/rootfs/boot/firmware.
15.25s: [22:14:54] Begin /home/ubuntu/bugg-os/pi-gen/stage2/01-sys-tweaks/01-run.sh
14.06s: mount: /dev/loop10p1 mounted on /home/ubuntu/bugg-os/pi-gen/work/BuggOS/export-image/rootfs/boot/firmware.
7.79s: [22:15:14] Begin /home/ubuntu/bugg-os/pi-gen/stage2/03-set-timezone/02-run.sh

real	11m19.860s
user	0m0.007s
sys	0m0.016s
```

Rob suggested running on ARM architecture, because pi-gen is using QEMU to virtualise execution of ARM binaries during the build. He recons that runing it natively might be quicker.

AWS offer ARM architecture (itself virtualised, but presumably this will still be a better option). 
[c6g.8xlarge](https://eu-north-1.console.aws.amazon.com/ec2/home?region=eu-north-1#InstanceTypeDetails:instanceType=c6g.8xlarge) 32 vCPUs arm64 64 GB
This is equivalent to my current [c5a.8xlarge](https://eu-north-1.console.aws.amazon.com/ec2/home?region=eu-north-1#InstanceTypeDetails:instanceType=c5a.8xlarge) instance.

Ok, I spun up a c6g.8xlarge instance, installed Ubuntu Jammy arm64, and copied over my home folder from the earlier, x86 instance.

I ran the build again:
Run 1:
```
Timings for printed lines:
16.15s: mount: /dev/loop5p1 mounted on /home/ubuntu/bugg-os/pi-gen/work/BuggOS/export-image/rootfs/boot/firmware.
15.82s: [21:16:12] Begin /home/ubuntu/bugg-os/pi-gen/export-image/prerun.sh
15.51s: I: Unpacking the base system...
11.50s: mount: /dev/loop5p1 mounted on /home/ubuntu/bugg-os/pi-gen/work/BuggOS/export-image/rootfs/boot/firmware.
6.93s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
6.65s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
5.83s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
5.64s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
5.42s: [21:14:50] Begin /home/ubuntu/bugg-os/pi-gen/stage-bugg-os/prerun.sh
4.96s: Duration:                 0.073168 seconds

real	5m40.282s
user	0m0.016s
sys	0m0.073s
```
Pretty decent!

Let's re-run it and see if it goes faster still:
Yup!
```
Timings for printed lines:
16.44s: [21:20:41] Begin /home/ubuntu/bugg-os/pi-gen/export-image/prerun.sh
13.65s: mount: /dev/loop5p1 mounted on /home/ubuntu/bugg-os/pi-gen/work/BuggOS/export-image/rootfs/boot/firmware.
11.74s: mount: /dev/loop5p1 mounted on /home/ubuntu/bugg-os/pi-gen/work/BuggOS/export-image/rootfs/boot/firmware.
6.62s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
6.51s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-v8
6.06s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
5.89s: update-initramfs: Generating /boot/initrd.img-6.1.0-rpi7-rpi-2712
4.91s: Duration:                 0.072533 seconds
4.79s: Duration:                 0.058625 seconds
4.04s: Fetched 25.0 MB in 3s (9657 kB/s)

real	1m50.215s
user	0m0.000s
sys	0m0.015s
```

Looks like the build is now disk IO-bound.
Chat GPT suggested changing from gp3 to io2. 
Build is now down to 1 min 11s.

If I change to a c7g series instance, the io2 will upgrade to io2 Block Express, which should be even faster. CPU's will be slightly faster too. I can use the opportunity to downsize the instance too, if I want, since I'm not using much in the way of CPU and RAM. Looks like 8 cores and 16GB would perform the same.

Increased to c7g.8xlarge:
Run 2:
real	0m56.821s

Very nice! I think I can reduce the instance CPU and RAM. Let's try 8 vCPU's and 16GB (c7g.2xlarge)
Run 2:
real	0m57.955s

c7g.medium (1 CPU, 2GB)
Run2:
real	2m13.138s

c7g.large (2 CPU, 4 GB)
Run 2:
real	1m53.010s

c7g.xlarge (4 CPU, 8 GB)
Run 2:
real	1m41.282s

c7g.2xlarge
Run 2:
real	0m57.184s

Yup, c7g.2xlarge is the optimum.

Reduce from 30000 IOPS to 3000:
No change in run time.
Reduce to 300 IOPS:
Fuck! You can only change a volume once every 6 hours!

300 IOPS:
Run 2:
real	2m16.727s

1000 IOPS:
Run 2:
real	1m9.243s
## Pricing
The instance:
c7g.2xlarge 0.3094 USD per Hour


# Building on AWS

Eventually (I currently believe) the build will run in a Github Actions runner (an arm64 one). I started on a GA x86 runner, but it was annoyling slow so I moved to AWS. On AWS I've been initiating the build with a shell script, but things are starting to diverge from how it will run in GA.

## Preparing `act`

`act` allows you to run a GA workflow locally, for the purposes of speeding up development.

### Install Docker
act depends on Docker, so let's install that per https://docs.docker.com/engine/install/ubuntu/ . We're running Ubuntu on the runner. 
* Use the apt install method.

### Install `act`
I can't remember doing it on my current AWS instance, but I must have used the curl method here:
https://github.com/nektos/act

### Supply a Github Token
https://github.com/nektos/act#github_token

## Running the build with `act`

```
sudo /home/ubuntu/bin/act -j image-builder -s GITHUB_TOKEN=(cat /home/ubuntu/github_token ) 
```

More complete build command:
```
git pull && sudo find deploy/ -type f -delete; time sudo /home/ubuntu/bin/act -j image-builder  -s GITHUB_TOKEN=`cat /home/ubuntu/github_token` --reuse && sudo ./copy_img_to_host.sh ; sudo ./delete_container.sh
```

## Debugging act

Occasionally, act will generate some really errors, almost like file corruption in the runner. You can fix it by running the build without --reuse, so the container is completely destroyed, but then you can't copy the build artefact out of the runner after act completes. 

## Build artefacts

Currently I use the copy_img_to_host.sh script to extract the built image artefact from the docker container, but there has to be a more elegant way to copy out the artefact - I think this is the way
https://github.com/nektos/act/issues/329#issuecomment-1187246629


### Connecting to the runner inside docker
```
ubuntu@ip-172-31-41-26:~/bugg-os$ sudo docker ps
CONTAINER ID   IMAGE                            COMMAND               CREATED         STATUS         PORTS     NAMES
f9cd28bcb7e6   catthehacker/ubuntu:act-latest   "tail -f /dev/null"   6 minutes ago   Up 6 minutes             act-Bugg-OS-Release-Build-image-builder-d3f30e8dfcb3ad150bb37b9559a0813eb2b29a4a054382de0ed5509625106a95
 
 sudo docker exec -it f9cd28bcb7e6 bash
 ```
 
 
## Downloading the built image (the "artefact")

Eventually the artefact will be pushed to Github's storage.
Currently, I can get act to run the actions/upload-artefact step, and it seems to upload the file *somewhere*, but I can't figure out where. Also, the upload takes ages. Kinda pointless since I have to download it to my local machine anyway.

I wrote a script copy_img_to_host.sh, which copies the built image out of the docker container and into a folder bugg-os/deploy on the AWS build server.

I spent a while buggering about with different methods to download.

### Preferred download and flash command

```
time curl --key ~/tools/soft/amazon-aws/aws-keys.pem scp://ubuntu@ec2-16-171-143-152.eu-north-1.compute.amazonaws.com/~/buggOS/pi-gen/deploy/Adze_0.2.0_alpha-21-gc90afef-lite.img.gz |gunzip  | sudo dd bs=10M of=/dev/sda && tada

```



## Streamed - curl with ssh:// | gunzip | dd

time curl --key ~/tools/soft/amazon-aws/aws-keys.pem scp://ubuntu@ec2-13-48-30-240.eu-north-1.compute.amazonaws.com/~/bugg-os/deploy/image_2024-01-24-BuggOS-lite.img.gz |gunzip  | sudo dd bs=10M of=/dev/sda
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  935M  100  935M    0     0  1537k      0  0:10:23  0:10:23 --:--:-- 1398k
100  935M  100  935M    0     0  1537k      0  0:10:23  0:10:23 --:--:-- 1537k
0+80281 records in
0+80281 records out
3376414720 bytes (3.4 GB, 3.1 GiB) copied, 694.819 s, 4.9 MB/s

________________________________________________________
Executed in  694.87 secs    fish           external
   usr time   59.74 secs    0.29 millis   59.74 secs
   sys time   19.12 secs    4.91 millis   19.11 secs

694 sec = 11.56

## Using the USB mass-storage-gadget

This is a MUCH faster way of flashing the Pi:

cd /home/jeff/projects/2022/bugg-v3/pi/rpiboot/mass-storage-gadget
sudo ../rpiboot -d . 


time curl --key ~/tools/soft/amazon-aws/aws-keys.pem scp://ubuntu@ec2-13-48-30-240.eu-north-1.compute.amazonaws.com/~/bugg-os/deploy/image_2024-01-30-BuggOS-lite.img.gz |gunzip  | sudo dd bs=10M of=/dev/sda && tada
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  837M  100  837M    0     0  7596k      0  0:01:52  0:01:52 --:--:-- 8634k
100  837M  100  837M    0     0  7596k      0  0:01:52  0:01:52 --:--:-- 7596k
0+102390 records in
0+102390 records out
3409969152 bytes (3.4 GB, 3.2 GiB) copied, 122.516 s, 27.8 MB/s

________________________________________________________
Executed in  122.58 secs    fish           external
   usr time   36.33 secs    0.36 millis   36.33 secs
   sys time   11.54 secs    3.05 millis   11.54 secs

# Scratchpad

sudo /home/ubuntu/bin/act -j image-builder --privileged -s GITHUB_TOKEN=(cat /home/ubuntu/github_token ) --reuse   
WARN[0000] --privileged is deprecated and will be removed soon, please switch to cli: `--container-options "--privileged"` or `.actrc`: `--container-options --privileged`. 

Error: failed to create container: 'Error response from daemon: invalid CapAdd: unknown capability: "CAP_[CAP_MKNOD]"'

CONTAINER ID   IMAGE                            COMMAND               CREATED         STATUS         PORTS     NAMES
90f0f65b7817   catthehacker/ubuntu:act-latest   "tail -f /dev/null"   4 minutes ago   Up 4 minutes             act-Bugg-OS-Release-Build-image-builder-d3f30e8dfcb3ad150


| [17:58:58] Begin /home/ubuntu/bugg-os/pi-gen/export-image/prerun.sh
| Creating loop device...
| mkdosfs: unable to open /dev/loop8p1: No such file or directory

| Creating loop device...
| mkdosfs: unable to open /dev/loop7p1: No such file or directory

Holy shit - this *exact issue* seems to have been fixed (at least in vanilla pi-gen - we're running it slightly out of spec by using its non-Docker mode in Docker...!) 
https://github.com/RPi-Distro/pi-gen/pull/741
I noticed that there's a difference between the version I'm looking at /home/ubuntu/bugg-os/pi-gen/export-image/prerun.sh line 49, and the new line 49!

But why is my clone out of datae...?
Ok, the change is in a fresh clone. Why not in the clone that act makes?

Ok fuck, the PR was merged into the x86 pi-gen, but not the arm64 branch!

Forked the repo, checked out the arm64 branch, cherry picked the merge commit, pushed back to my own fork.

[Bugg OS Release Build/image-builder] 🏁  Job succeeded

________________________________________________________
Executed in  256.80 secs      fish           external
   usr time   24.89 millis  166.00 micros   24.72 millis
   sys time  115.54 millis  166.00 micros  115.37 millis

Same with --reuse

[Bugg OS Release Build/image-builder]   ❗  ::error::Unable to get ACTIONS_RUNTIME_TOKEN env variable

Soundcard kernel module:

| make -C /lib/modules/6.2.0-1018-aws/build M=/opt/bugg-soundcard-driver/codec modules
| make[1]: *** /lib/modules/6.2.0-1018-aws/build: No such file or directory.  Stop.



## Adjust serial terminal geometry
bugg@bugg-v5:~$ stty rows 33 cols 150

7d7f5ebd7f5608ca4e61133fd35ae9a3

