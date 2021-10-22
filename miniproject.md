# Mini Project
this porject is some tasks for chromium OS.

## Connectivity, tools & infrastructure readiness
### Linux distribution
My PC OS is exactly Ubuntu 18.04.

###  Connectivity
I use qv2ray app to access google things.  

And add http_proxy & https_proxy into ~/.bashrc  

 ```shell
 echo -e 'export http_proxy=http://127.0.0.1:8889\nexport http_proxy=https://127.0.0.1:8889' >> ~/.bashrc && source ~/.bashrc
 ```
 
## Get Chromium OS code and start building
### Prerequisites
#### python
Since the chrominum dev guide shows that python 3.6 or higher is required, we should check the python verison.  
```shell
python -v
```    
My python links to python2 which version is python 2.7.15.       
So I remove this symbol link and recreate a symbol link to point to python3.    
```shell
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
```shell
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git path/to/chromiumos/src/chromium/depot_tools
```
Then add the folder into PATH.  
```shell
echo "export PATH=/path/to/depot_tools:$PATH" >> ~/.bashrc && source ~/.bashrc
```

#### Tweak your sudoers configuration
The solution metioned in guide cannot work properly.  
It will fail with error log as below:  
<pre>visudo: /etc/sudoers.d/relax_requirements.tmp 未更改
</pre>  
We can add the content manually.  
```shell
sudo vi /etc/sudoers.d/relax_requirements
```
And add below content:  
<pre>Defaults !tty_tickets
Defaults timestamp_timeout=180
</pre>  

#### Set locale
```shell
sudo apt-get install locales
sudo dpkg-reconfigure locales
```

#### Configure git
If you use github or participate some project using git to control their version, you will already config them.  
```shell
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

### Get the Source
#### make dir to save source code
I want to download the source code into my  NVME SSD, which is faster than HDD.  
I already added the mount info into /etc/fstab, it will mount on   /home/pyy/NVME1T.  
```shell
mkdir -p /home/pyy/NVME1T/chromium
```
Maybe you can make dir as you want.

#### Get the source code
I used the repo command before, so I can run the command.   
This cmd also exists in depot_tools.  
```shell
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
```shell
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

#### enter chroot
`cros_sdk --enter`

#### Select a board
set env variable in chroot.    
```shell
export BOARD=amd64-generic
```  
Then run setup board cmd.  
```shell
setup_board --board=amd64-generic
```
#### Set  passwd  
```shell
./set_shared_user_password.sh
```
#### Build all packages  
```shell
./build_packages --board=${BOARD}
```  
**It costs long time to build.......And there is no logs to show the progress.**    
**Solution:**  
We can open another terminal and enter chroot. Then find the component which is building right now.  
I found that building `chromeos-chrome-92.0.4515.162_rc-r1` will cost long time.    
```shell
cros_sdk --enter
tail -f /build/amd64-generic/tmp/portage/logs/chromeos-base\:chromeos-chrome-92.0.4515.162_rc-r1\:202110xxxxxx.log
```
It shows the appending logs as below, we can easily figure out the progress.
<pre>
[50631/77688] CXX obj/content/browser/browser/background_fetch_data_manager.o
</pre>






