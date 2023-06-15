# Modified webui-user.sh and webui.sh to make this work for jetson agx orin
Changed user setting to use local pytorch install that works for nvidia jetson agx orin  

I removed torchvision but did not replace it with anything and I think I may regret that.

https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048/13  

# specific version of torch that was build for jetson nano  
build instructions at bottom of first post:  
https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048  

git clone --recursive --branch <version> http://github.com/pytorch/pytorch
cd pytorch

Set Build Options

$ export USE_NCCL=0
$ export USE_DISTRIBUTED=0                    # skip setting this if you want to enable OpenMPI backend
$ export USE_QNNPACK=0
$ export USE_PYTORCH_QNNPACK=0
$ export TORCH_CUDA_ARCH_LIST="5.3;6.2;7.2"   # or "7.2;8.7" for JetPack 5 wheels for Xavier/Orin

$ export PYTORCH_BUILD_VERSION=<version>  # without the leading 'v', e.g. 1.3.0 for PyTorch v1.3.0
$ export PYTORCH_BUILD_NUMBER=1

sudo apt-get install python3.9-dev
pip3 install -r requirements.txt
pip3 install scikit-build
pip3 install ninja
LD_DEBUG=all make
python3.9 setup.py sdist bdist_wheel

# torchvision
At the bottom of that link there is a set of instructions for torchvision
$ git clone https://github.com/pytorch/vision
$ cd vision
python3.9 -m venv venv
source ./venv/bin/activate
pip install torch
pip install wheel
python3.9 setup.py sdist bdist_wheel

https://github.com/pytorch/vision   

# TCMalloc install  

To install TCMalloc:

sudo apt-get install google-perftools

To replace allocators in system-wide manner I edit /etc/environment (or export from /etc/profile, /etc/profile.d/*.sh):

echo "LD_PRELOAD=/usr/lib/libtcmalloc.so.4" | tee -a /etc/environment

To do the same in more narrow scope you can edit ~/.profile, ~/.bashrc, /etc/bashrc, etc.

https://stackoverflow.com/questions/29205141/how-to-use-tcmalloc  

# Install JetPack Components

Once the initial setup is complete, you can install the latest JetPack components that correspond to your L4T version from the Internet.

Open a terminal window if you are on Ubuntu desktop ( Ctrl+Alt+T ).

Attention

Check your L4T version first to see if you have a unit flashed with older version of the BSP.

cat /etc/nv_tegra_release

You may get something like this, # R34 (release), REVISION: 1.0, GCID: 30102743, BOARD: t186ref, EABI: aarch64, DATE: Wed Apr 6 19:11:41 UTC 2022, and this shows that you have L4T for JetPack 5.0 Developer Preview.

If you have an earlier version of L4T, issue the following command to manually put the apt repository entries using commands below.

sudo bash -c 'echo "deb https://repo.download.nvidia.com/jetson/common r34.1 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list'
sudo bash -c 'echo "deb https://repo.download.nvidia.com/jetson/t234 r34.1 main" >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list'

If you see R34 (release), REVISION: 1.0 or newer, then your apt sources lists are already up to date and you can proceed

Issue the following commands to install JetPack components.

sudo apt update
sudo apt dist-upgrade
sudo reboot
sudo apt install nvidia-jetpack

It can take about an hour to complete the installation (depending on the speed of your Internet connection).

https://developer.nvidia.com/embedded/learn/get-started-jetson-agx-orin-devkit#step-2-install-jetpack-components  


# gfpgan hangs on install for some reason?

https://www.reddit.com/r/StableDiffusion/comments/y9dn3w/roughly_how_long_should_it_take_for_gfpgan_to/  

https://github.com/grpc/grpc/issues/24390

Add the following:

export GRPC_PYTHON_BUILD_SYSTEM_OPENSSL=1
export GRPC_PYTHON_BUILD_SYSTEM_ZLIB=1

by:

export GRPC_PYTHON_BUILD_SYSTEM_OPENSSL=1
export GRPC_PYTHON_BUILD_SYSTEM_ZLIB=1

# libraries that arent present while building torchvision - installing libpng and libjpeg 
UserWarning: Failed to load image Python extension: ''If you don't plan on using image functionality from `torchvision.io`, you can ignore this warning. Otherwise, there might be something wrong with your environment. Did you have `libjpeg` or `libpng` installed before building `torchvision` from source?  

sudo apt-get install libpng-dev  
sudo apt-get install libjpeg-dev

sudo apt-get install zlib1g-dev

# 'No module 'xformers'. Proceeding without it.' error in launch

set COMMANDLINE_ARGS=--reinstall-xformers --xformers  

source: https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/5303#discussioncomment-4751796  

# changing versions of python

apt search python3 | grep "python3\."
sudo apt install python3.9
python3.9 -V
python3.9 -m venv "my_env_name"

# downloading specific version
python3.9 -m pip download --only-binary :all: --dest ./to39 --no-cache torch  
torch is the name of the package, and the python3.9 makes sure its for version 3.9 of python, to39 is the directory