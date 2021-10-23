# Content  
[Mini Project](#mini-project)  
　1. [Connectivity, tools & infrastructure readiness](#connectivity,-tools-&-infrastructure-readiness)  
　　1.1 [Linux distribution](#linux-distribution)  
　　1.2 [Connectivity](#connectivity)  
　2. [Get Chromium OS code and start building](#get-chromium-os-code-and-start-building)  
　　2.1 [Prerequisites](#prerequisites)  
　　　　2.1.1 [python](#python)  
　　　　2.1.2 [Install development tools](#install-development-tools)  
　　　　2.1.3 [Install depot_tools](#install-depot_tools)  
　　　　2.1.4 [Tweak your sudoers configuration](#tweak-your-sudoers-configuration)  
　　　　2.1.5 [Set locale](#set-locale)  
　　　　2.1.6 [Configure git](#configure-git)  
　　2.2 [Get the Source](#get-the-source)  
　　　　2.2.1 [make dir to save source code](#make-dir-to-save-source-code)  
　　　　2.2.2 [Get the source code](#get-the-source-code)  
　　　　2.2.3 [Optionally add Google API keys](#optionally-add-google-api-keys)  
　　　　2.2.4 [Make sure you are authorized to access Google Storage (GS) buckets](#make-sure-you-are-authorized-to-access-google-storage-gs-buckets)  
　　2.3 [Building Chromium OS](#building-chromium-os)  
　　　　2.3.1 [Create a chroot](#create-a-chroot)  
　　　　2.3.2 [Enter chroot](#enter-chroot)  
　　　　2.3.3 [Select a board](#select-a-board)  
　　　　2.3.4 [Set  passwd](#set--passwd)  
　　　　2.3.5 [Build all packages](#build-all-packages)  
　　　　2.3.6 [Build test image](#build-test-image)  
　　　　2.3.7 [Convert test image to VM usable image](#convert-test-image-to-vm-usable-image)  
　　　　2.3.8 [run chromium os in vm](#run-chromium-os-in-vm)  
　　　　2.3.9 [VNC & SSH to login](#vnc-&-ssh-to-login)  
　3. [Kernel Replacement](#kernel-replacement)  
　　3.1 [Update the kernel on a live running ChromiumOS instance.](#update-the-kernel-on-a-live-running-chromiumos-instance)  
　　　　3.1.1 [cros_workon_make to build kernel](#cros_workon_make-to-build-kernel)  
　　　　3.1.2 [emerge to build kernel](#emerge-to-build-kernel)  
　　　　3.1.3 [update kernel](#update-kernel)  
　　3.2 [Replace kernel ref things in Image](#replace-kernel-ref-things-in-image)  
　4. [CrOS devserver in docker](#cros-devserver-in-docker)  
　　4.1 [Build Docker image](#build-docker-image)  
　　　　4.1.1 [install docker](#install-docker)  
　　　　4.1.2 [build docker image](#build-docker-image)  
　　　　　　4.1.2.1 [make docker_devserver folder](#make-docker_devserver-folder)  
　　　　　　4.1.2.2 [Dockerfile](#dockerfile)  
　　　　　　4.1.2.3 [build devserver image](#build-devserver-image)  
　　　　4.1.3 [run docker image](#run-docker-image)  
　5. [Connecting the dots](#connecting-the-dots)  
　　5.1 [Modify the  /etc/lsb-release](#modify-the--etclsb-release)  
　　5.2 [Generate an update payload](#generate-an-update-payload)  
　　5.3 [Move the update payload to the  static_dir of the devserver inside docker](#move-the-update-payload-to-the--static_dir-of-the-devserver-inside-docker)  
　　5.4 [Perform  update_engine_client --update in your Chromium OS instance](#perform--update_engine_client---update-in-your-chromium-os-instance)  
　　　　5.4.1 [run `update_engine_client --update`](#run-`update_engine_client---update`)  
　　　　5.4.2 [check for updates in chromium os settings](#check-for-updates-in-chromium-os-settings)  
　6. [Thinking](#thinking)  
　　6.1 [About google api keys](#about-google-api-keys)  
　　6.2 [OTA](#ota)  
  
# Mini Project
this porject is some tasks for chromium OS.

## Connectivity, tools & infrastructure readiness
### Linux distribution
My PC OS is exactly Ubuntu 18.04.

###  Connectivity
I use qv2ray app to access google things.  

And add http_proxy & https_proxy into ~/.bashrc  

 ```sh
 echo -e 'export http_proxy=http://127.0.0.1:8889\nexport http_proxy=https://127.0.0.1:8889' >> ~/.bashrc && source ~/.bashrc
 ```
 
## Get Chromium OS code and start building
### Prerequisites
#### python
Since the chrominum dev guide shows that python 3.6 or higher is required, we should check the python verison.  
```sh
python -v
```    
My python links to python2 which version is python 2.7.15.       
So I remove this symbol link and recreate a symbol link to point to python3.    
```sh
sudo rm /usr/bin/python && sudo ln -s /usr/bin/python3 /usr/bin/python
```
Then run `python -v`, the result shows as below:    
<pre> pyy@五 10月 22 23:01 chromium$ python  
Python 3.6.9 (default, Jan 26 2021, 15:33:00)   
[GCC 8.4.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>></pre>
**Or you can use `pyenv` to config the global python version as python3**  

#### Install development tools
<pre>sudo add-apt-repository universe
sudo apt-get install git gitk git-gui curl xz-utils \
     python3-pkg-resources python3-virtualenv python3-oauth2client</pre>
 
#### Install depot_tools
I found that there is the same name folder in chromium src code.  
And I compared these two folders found that cloned one using python3 but source code one using python.      
Maybe run clone command to replace the one in source code(if you do not recreate python symbol link).  
```sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git path/to/chromiumos/src/chromium/depot_tools
```
Then add the folder into PATH.  
```sh
echo "export PATH=/path/to/depot_tools:$PATH" >> ~/.bashrc && source ~/.bashrc
```

#### Tweak your sudoers configuration
The solution metioned in guide cannot work properly.  
It will fail with error log as below:  
<pre>visudo: /etc/sudoers.d/relax_requirements.tmp 未更改
</pre>  
We can add the content manually.  
```sh
sudo vi /etc/sudoers.d/relax_requirements
```
And add below content:  
<pre>Defaults !tty_tickets
Defaults timestamp_timeout=180
</pre>  

#### Set locale
```sh
sudo apt-get install locales
sudo dpkg-reconfigure locales
```

#### Configure git
If you use github or participate some project using git to control their version, you will already config them.  
```sh
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

### Get the Source
#### make dir to save source code
I want to download the source code into my  NVME SSD, which is faster than HDD.  
I already added the mount info into /etc/fstab, it will mount on   /home/pyy/NVME1T.  
```sh
mkdir -p /home/pyy/NVME1T/chromium
```
Maybe you can make dir as you want.

#### Get the source code
I used the repo command before, so I can run the command.   
This cmd also exists in depot_tools.  
```sh
cd /home/pyy/NVME1T/chromium
repo init -u https://chromium.googlesource.com/chromiumos/manifest -b release-R92-13982.B
repo sync -j4
```
It will take a long time to sync code. 

#### Optionally add Google API keys
You can add API & OAuth 2.0 client keys with steps shows in [API keys](https://www.chromium.org/developers/how-tos/api-keys)  
**Be noticed that there is no Ohter application type in OAuth 2.0 client credentials, you can use desktop app type instead.**  
Add below content into `args.gn` in chroot env.  
<pre>
google_api_key = "your_api_key"
google_default_client_id = "your_client_id"
google_default_client_secret = "your_client_secret"
</pre>
Change the related keys into your keys created before.  
Then run command `gn args out/your_out_dir_here`.  

#### Make sure you are authorized to access Google Storage (GS) buckets
At first we should run command to upgrade the `gsutil`.  
```sh
path/to/chromiumos/chromite/scripts/gsutil
echo "export PATH=path/to/chromiumos/chromite/scripts:$PATH" >> ~/.bashrc && source ~/.bashrc
```
Then we can run `gsutil config` to config the project-id and generate ~/.boto.  
Here I met a error, it always timeout when copy the authorization code.  
  **Quetion:**
<pre>
Unable to connect to accounts.google.com during OAuth2 flow. This can
happen if your site uses a proxy. If you are using gsutil through a
proxy, please enter the proxy's information; otherwise leave the
following fields blank.
</pre>  
  **Solution:**  
The timeout is caused by proxy server. So we sould set the proxy info.  
<pre>
What is your proxy host? 127.0.0.1
What is your proxy type (socks4, socks5, http)? http
What is your proxy port? 8889
What is your proxy user (leave blank if not used)? 
What is your proxy pass (leave blank if not used)? 
Should DNS lookups be resolved by your proxy? (Y if your site disallows client DNS lookups; NOT supported for socks)? N
</pre>
After readd the authorization code, it will ask you to set the project id: xxx.  
At last it generate the ~/.boto file used by gsutil.  

### Building Chromium OS
#### Create a chroot
Run `cros_sdk`to download and create a chroot environment.  
But I met an curl error.    
**Question:**    
![cros_sdk curl error](/assets/img/cros_sdk_curl_error.png "cros_sdk curl error")    
**Solution:**  
I download the [tar](https://storage.googleapis.com/chromiumos-sdk/cros-sdk-2021.05.18.170403.tar.xz) file manually and copy it into `path/to/chromiumos/.cache/sdks/` folder.  
Then run `cros_sdk --create` instead.    
It also takes a long time to create the chroot env.  

#### Enter chroot
`cros_sdk --enter`  
After executed cmd above, the `$PS1` should be `(cr) ((f7abe57...)) pyy@pyy-U ~/trunk/src/scripts $`.  


#### Select a board
set env variable in chroot.    
```sh
export BOARD=amd64-generic
```  
Then run setup board cmd.  
```sh
setup_board --board=amd64-generic
```
#### Set  passwd  
```sh
./set_shared_user_password.sh
```  
You can set the password as you want.  

#### Build all packages  
```sh
./build_packages --board=${BOARD}
```  
**It costs long time(12+hrs) to build.......And there is no logs to show the progress.**   
![cost time](/assets/img/cost_time.png "cost time")    
![full time](/assets/img/full_time.png "full time")   
**Solution:**  
We can open another terminal and enter chroot.   
Then find the component which is building right now.  
I found that building `chromeos-chrome-92.0.4515.162_rc-r1` will cost long time.    
```sh
cros_sdk --enter
tail -f /build/amd64-generic/tmp/portage/logs/chromeos-base\:chromeos-chrome-92.0.4515.162_rc-r1\:202110xxxxxx.log
```
It shows the appending logs as below, we can easily figure out the progress.
<pre>
[50631/77688] CXX obj/content/browser/browser/background_fetch_data_manager.o
</pre>
`./build_packages --help` shows that the fastest builds, use `--nowithautotest --noworkon`.  
And `--jobs`  means how many packages to build in parallel at maximum.   Maybe we can set it as a proper value to speed up the build process.



#### Build test image
Once the `build_packages` step is finished, you can build a Chromium OS-base developer test image by running the command below
```sh
./build_image --board=${BOARD} --noenable_rootfs_verification test
```
**If building a test image, the password set using `set_shared_user_password.sh` will be replaced by  `test0000`**  
The `--noenable_rootfs_verification` turns off verified boot allowing you to freely modify the root file system.   
The image will be built in `/path/to/chromiumos/src/build/images/${BOARD}}/latest/`  
![image](/assets/img/image.png "image")   
![latest](/assets/img/latest.png "latest folder")   

#### Convert test image to VM usable image
The log above shows that we can convert test image to VM image use below command.  
```sh
./image_to_vm.sh --from=../build/images/amd64-generic/R92-13982.89.2021_10_23_0814-a1 --board=amd64-generic --test
```
We can find the `image_to_vm.sh` script in `~/trunk/src/scripts`.
```sh
./image_to_vm.sh --help
USAGE: ./image_to_vm.sh [flags] args
flags:
  --adjust_part:  Adjustments to apply to the partition table (default: '')
  --board:  Board for which the image was built (default: '')
  --from:  Directory containing rootfs.image and mbr.image (default: '')
  --disk_layout:  The disk layout type to use for this image. (default: '2gb-rootfs-updatable')
  -chromiumos_image.bin.,--[no]test_image:  Acquires image from chromiumos_test_image.bin instead of  (default: false)
  --to:  Destination folder for VM output file(s) (default: '')
  -h,--[no]help:  show this help (default: false)
```
We know the flag `--test_image` will use the image `chromiumos_test_image.bin` instead of `chromiumos_image.bin`  
So we can also run command as below:
```sh
./image_to_vm.sh --board=${BOARD}  --test_image
```  
The image will be built in `/path/to/chromiumos/src/build/images/${BOARD}}/latest/`  
![vm image](/assets/img/vm_image.png "vm image")   


#### run chromium os in vm
As the log shows that we can run vm image with below command.  
```sh
cros_vm --start --board $BOARD --image-path \
/mnt/host/source/src/build/images/${BOARD}}/latest/chromiumos_test_image.bin
```
![vm start](/assets/img/vm_start.png "vm start")   

If you do not set args.gn before building image, you can add Google API keys during Runtime.
```sh
sudo mount -o remount,rw /
vi /etc/chrome_dev.conf
```
Add below content.  
<pre>
GOOGLE_API_KEY=your_api_key
GOOGLE_DEFAULT_CLIENT_ID=your_client_id
GOOGLE_DEFAULT_CLIENT_SECRET=your_client_secret
</pre>
You can run `cros_vm --stop` to stop the chromium os.  


#### VNC & SSH to login
**SSH**
```sh
$ ssh -p 9222 chronos@127.0.0.1 
chronos@localhost ~ $ uname -r
4.14.232-17774-gdfbdec1b1a0b
```
![ssh](/assets/img/ssh.png "ssh login")   


**VNC**  
In Ubuntu there is an app named Remmina, we can create a VNC link in it.  
Just configure the IP and user/passwd.
![vnc config](/assets/img/vnc_config.png "vnc config")  
![vnc](/assets/img/vnc.png "vnc login")  

## Kernel Replacement
### Update the kernel on a live running ChromiumOS instance.
Using `update_kernel.sh`  script, we can update the kernel on a live running Chromium OS instance.  
#### cros_workon_make to build kernel
run command as below to make an incremental build of the kernel.  
```sh
cros-workon-${BOARD} start chromeos-kernel-5_10
FEATURES="noclean" cros_workon_make --board=${BOARD} --install chromeos-kernel-5_10
```
It happens an error like below.  
<pre>
>>> 10:03:32 Failed to install sys-kernel/chromeos-kernel-5_10-9999 to /build/amd64-generic/, Log file:
>>> 10:03:32   /build/amd64-generic/tmp/portage/logs/sys-kernel:chromeos-kernel-5_10-9999:20211023-014314.log
</pre>
Check the build log, I find there is a  file collisions.  
<pre>
^[[31;01m * ^[[39;49;00mDetected file collision(s):
^[[31;01m * ^[[39;49;00m
^[[31;01m * ^[[39;49;00m        /build/amd64-generic/usr/lib/debug/boot/vmlinux
^[[31;01m * ^[[39;49;00m        /build/amd64-generic/boot/vmlinuz
^[[31;01m * ^[[39;49;00m
</pre>    
**Solution:**  
It should run unmerge command to remove old kernel things.  
```sh
emerge-${BOARD} --unmerge chromeos-kernel-4_14
```
Then re-run the `FEATURES="noclean" cros_workon_make --board=${BOARD} --install chromeos-kernel-5_10` command

#### emerge to build kernel
We can use `emerge` command as instead to make kernel.  But it's slower than `cros_workon_make`.
```sh
emerge-${BOARD} --nodeps chromeos-kernel-[x_y]
```
#### update kernel
Run below command. 
Make sure the chromium OS already start.  
```sh
./update_kernel.sh --board=${BOARD} --remote=127.0.0.1 --ssh_port=9222
```
![update_kernel](/assets/img/update_kernel.png "update kernel")  
![result](/assets/img/result.png "result")  

### Replace kernel ref things in Image
Since we build image with `--noenable_rootfs_verification`, we can modify the image things.  
So there is a method to replace kernel like Android ROM to replace the boot.img.  
TODO

## CrOS devserver in docker
**Note: Every time you create a Chromium OS build, the URL of the dev server corresponding to your development machine is put into /etc/lsb-release.**  
**This file can be changed post-installation to manage a test machine's update source or can be overridden in /mnt/stateful_partition/etc/lsb-release**  
### Build Docker image
#### install docker
```sh
sudo snap install docker
sudo apt  install docker.io
docker
```

#### build docker image
As guide said we can run `start_devserver` in chroot to start devserver.  
So if we want to run it in docker image, we should know what it do.    
So we run `which start_devserver` to get its path `/usr/bin/start_devserver`.   
In fact it calls `/usr/lib/devserver/devserver.py` directly.  
We run it in chroot, then we access it via `http://localhost:8080/`.  
![devserver](/assets/img/devserver.png "devserver")  
So we should get the devserver.py and its dependacies and build a docker image based on ubuntu.  
```sh
pyy@六 10月 23 11:04 src$ find ./ -name "devserver.py"
./platform/dev/devserver.py
```
Enter into `src/platform/dev/`, check the `.git/config`.  
<pre>
[core]
        repositoryformatversion = 0
        filemode = true
[filter "lfs"]
        smudge = git-lfs smudge --skip -- %f
        process = git-lfs filter-process --skip
[remote "cros"]
        url = https://chromium.googlesource.com/chromiumos/platform/dev-util
        review = https://chromium-review.googlesource.com
        projectname = chromiumos/platform/dev-util
        fetch = +refs/heads/*:refs/remotes/cros/*
[gc]
        autoDetach = false
</pre>
Now I run the `devserver.py` locally, it will raise error.  
<pre>
Traceback (most recent call last):
  File "./devserver.py", line 43, in <module>
    import cherrypy
ModuleNotFoundError: No module named 'cherrypy'
</pre>
So the lib `cherrypy` is needed, we run `sudo pip3 install cherrypy` to install it.  
re-run `devserver.py`, it can start successfully.  
Now I should design the Dockerfile and create docker image.
##### make docker_devserver folder
```sh
mkdir ~/docker_devserver
touch Dockerfile
cp -rf /path/to/chromiumos/src/platform/dev dev
cp -rf /path/to/chromiumos/chromite chromite
```
##### Dockerfile
```Dockerfile
# This dockerfile uses the ubuntu image
# Usage: run devserver

# Base image to use
FROM ubuntu:18.04

MAINTAINER sl0v3c pyy101727@gmail.com

# Commands to update the image
RUN apt-get update && apt-get install -y python3 python-pip curl python3-distutils
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
RUN python3 get-pip.py
RUN pip3 install -i https://mirrors.ustc.edu.cn/pypi/web/simple pip -U
RUN pip3 config set global.index-url https://mirrors.ustc.edu.cn/pypi/web/simple
RUN pip3 install cherrypy

RUN mkdir dev_server

COPY ./dev  /dev_server/dev
COPY ./chromite  /dev_server/dev/chromite

RUN useradd -ms /bin/bash test
RUN chown -R test:test /dev_server/

USER test

WORKDIR /dev_server/dev
ErrorCode::kNoUpdate(53)
CMD ./devserver.py
```

##### build devserver image
```sh
cd ~/docker_devserver
sudo docker build -t dev_server .
```

#### run docker image
```sh
sudo docker run -p 8888:8080 --rm -ti dev_server
```
![docker run](/assets/img/docker_run.png "docker run")  
![docker](/assets/img/docker.png "docker")  

## Connecting the dots
### Modify the  /etc/lsb-release 
```sh
sudo mount -o remount, rw /
sudo vim /etc/lsb-release
```
The content as below.    
The IP can be got by exec `ifconfig` in host.    
Because of docker dev_server image mapped port to host 8888. 
<pre>
CHROMEOS_RELEASE_NAME=Chromium OS
CHROMEOS_AUSERVER=http://IP:8888/update
CHROMEOS_DEVSERVER=http://IP:8888
CHROMEOS_RELEASE_KEYSET=devkeys
CHROMEOS_RELEASE_TRACK=testimage-channel
CHROMEOS_RELEASE_BUILD_TYPE=Test Build - pyy
CHROMEOS_RELEASE_DESCRIPTION=13982.89.2021_10_23_0814 (Test Build - pyy) developer-build amd64-generic
CHROMEOS_RELEASE_BOARD=amd64-generic
CHROMEOS_RELEASE_BRANCH_NUMBER=89
CHROMEOS_RELEASE_BUILD_NUMBER=13982
CHROMEOS_RELEASE_CHROME_MILESTONE=92
CHROMEOS_RELEASE_PATCH_NUMBER=2021_10_23_0814
CHROMEOS_RELEASE_VERSION=13982.89.2021_10_23_0814
GOOGLE_RELEASE=13982.89.2021_10_23_0814
</pre>

Then restart chromium os and ssh login it.  
Run command as below in chromium os instance to check it can connected the docker devserver or not.  
```sh
curl http://172.17.0.1:8888
```
It will show the html content like this.  
<pre>
Welcome to the Dev Server!<br>
<br>
Here are the available methods, click for documentation:<br>
<br>
<a href=doc/build>build</a><br>
<a href=doc/check_health>check_health</a><br>
<a href=doc/controlfiles>controlfiles</a><br>
<a href=doc/is_staged>is_staged</a><br>
<a href=doc/latestbuild>latestbuild</a><br>
<a href=doc/list_image_dir>list_image_dir</a><br>
<a href=doc/list_suite_controls>list_suite_controls</a><br>
<a href=doc/locate_file>locate_file</a><br>
<a href=doc/setup_telemetry>setup_telemetry</a><br>
<a href=doc/stage>stage</a><br>
<a href=doc/symbolicate_dump>symbolicate_dump</a><br>
<a href=doc/update>update</a><br>
<a href=doc/xbuddy>xbuddy</a><br>
<a href=doc/xbuddy_capacity>xbuddy_capacity</a><br>
<a href=doc/xbuddy_translate>xbuddy_translate</a>
</pre>

### Generate an update payload  
Rebuild a test image. Before exec it, should emerge --unmerge the kernel 5.10.   
Or it will fail because of file collisions.
```sh
emerge-${BOARD} --unmerge chromeos-kernel-5_10
./build_packages --board=${BOARD} --nowithautotest --noworkon --jobs 8 --autosetgov
./build_image --board=${BOARD} --noenable_rootfs_verification test
```
Then we should use `cros_sdk` to generate the payload.
```sh
cros_generate_update_payload  --tgt-image ../../backup/amd64-generic/latest/chromiumos_test_image.bin --output ../build/
```
it will generate delta.bin and bulid.json, copy them into `static_dir`.

### Move the update payload to the  static_dir of the devserver inside docker  
Before copy the json file, should add the appid which is null will cause error. 
```json
{
	"appid": "{87efface-864d-49a5-9bb3-4b050a7c227a}",
	"is_delta": false,
	"metadata_signature": null,
	"metadata_size": 57890,
	"sha256_hex": "zyYEFWJU5NDuNXaz7XGkJHUNXadGKsDJlvAmOj5WoMY=",
	"size": 641655114,
	"target_version": "99999.1.1",
	"version"
}
```
I just use docker cp to copy all the images to docker container:/path/.  
```sh
sudo docker cp delta.bin.json 71d1ff71a2fa:/dev_server/dev/static/amd64-generic/latest
sudo docker cp delta.bin 71d1ff71a2fa:/dev_server/dev/static/amd64-generic/latest
```
Now we can download the bin to check the GET api.
![download](/assets/img/download.png "download")  

### Perform  update_engine_client --update in your Chromium OS instance  
#### run `update_engine_client --update`
```sh
chronos@localhost ~ $ update_engine_client --update --omaha_url=172.17.0.1:8080/update/
2021-10-23T16:17:24.844362Z INFO update_engine_client: [update_engine_client.cc(511)] Forcing an update by setting app_version to ForcedUpdate.
2021-10-23T16:17:24.844561Z INFO update_engine_client: [update_engine_client.cc(513)] Initiating update check.
2021-10-23T16:17:24.846279Z INFO update_engine_client: [update_engine_client.cc(542)] Waiting for update to complete.
```
run this command will cause many error.
<pre>
ErrorCode::kOmahaErrorInHTTPResponse(37)
ErrorCode::kDownloadTransferError(9)
ErrorCode::kInstallDeviceOpenError(7)
ErrorCode::kNoUpdate(53)
</pre>
I think that 53 caused by no update and the response code is 200, it's OK!

#### check for updates in chromium os settings
![updating](/assets/img/updating.png "updating")  
![up to date](/assets/img/up_to_date.png "up to date")  

## Thinking
### About google api keys
I think it should design a config center like Apollo to configure the api keys.  
The api keys can be only one used by all the apps or services. 
It also can be many different vendor to different apps & services.  

###  OTA
The OTA thing should dig deeper.  
The arch seems easy, just a server and provides api. You can access via RPC or http.  
But there is no documents or any clue.  
Just trace the update flow and figure the logic out.  
Maybe the ota can be easier by design new arch and use the new tech in internet scope.
We can collect the core api chromium os using now and re-implement a new service with gateway & ms.
Or use websocket to do the pub/sub/push/pull things.













