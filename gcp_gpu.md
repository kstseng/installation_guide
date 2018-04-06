## Note for GCP GPU

Tutorial: [在 Google Cloud Platform 上使用 GPU 和安裝深度學習相關套件](https://medium.com/@kstseng/%E5%9C%A8-google-cloud-platform-%E4%B8%8A%E4%BD%BF%E7%94%A8-gpu-%E5%92%8C%E5%AE%89%E8%A3%9D%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92%E7%9B%B8%E9%97%9C%E5%A5%97%E4%BB%B6-1b118e291015)

### Outline

* [SSH to Instance](#ssh-to-instance)
* [Install CUDA](#install-cuda)
* [Install cuDNN](#install-cudnn)
* [Install Anaconda](#install-anaconda)
* [Create venv](#create-venv)
* [Install Tensorflow](#install-tensorflow)
* [Running Jupyter notebooks in your local computer](#running-jupyter-notebooks-in-your-local-computer)

#### SSH to Instance

* initialization

```bash
## local

gcloud init
```
* ssh to instance

```bash
## local

gcloud compute ssh instance-1
```

* copy `sth` to remote, ex: cudnn-xxx

```bash
## local

gcloud compute scp cudnn-8.0-linux-x64-v7.1.tgz instance-1:/home/davidtseng
```

---

#### Install CUDA

```bash
## remote

# get driver
curl -O https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1604_8.0.61-1_amd64.deb

# update to cuda 9.0
sudo apt-get update
sudo apt-get install cuda-9-0
sudo nvidia-smi -pm 1
sudo nvidia-smi -ac 2505,875 # performance optimziation from google suggestion

# add to environment variable
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
echo 'export PATH=$PATH:$CUDA_HOME/bin' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$CUDA_HOME/lib64' >> ~/.bashrc
source ~/.bashrc
nvidia-smi
```

---

#### Install cuDNN

```bash
## local

# scp package from local to remote
gcloud compute scp cudnn-9.0-linux-x64-v7.tgz instance-1:/home/davidtseng
```


```bash
## remote

tar xzvf cudnn-9.0-linux-x64-v7.tgz
sudo cp cuda/lib64/* /usr/local/cuda/lib64/
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
rm -rf ~/cuda
rm cudnn-9.0-linux-x64-v7.tgz
```

---

#### Install Anaconda

[reference](https://www.digitalocean.com/community/tutorials/how-to-install-the-anaconda-python-distribution-on-ubuntu-16-04)

```bash
## remote

curl -O https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh
sha256sum Anaconda3-5.0.1-Linux-x86_64.sh
bash Anaconda3-5.0.1-Linux-x86_64.sh
source ~/.bashrc
conda list
```

* if `conda: command not found`, add environment variable to ~/.bashrc manually.
	* `export PATH=/home/davidtseng/anaconda3/bin:$PATH`

---

#### Create venv

```bash
conda create --name py36 python=3.6 anaconda
source activate py36
```

---

#### Install Tensorflow

```bash
pip install --upgrade tensorflow-gpu
```

* Test gpu work or not

```python
# In python interactive mode

import tensorflow as tf
from tensorflow.python.client import device_lib
print(device_lib.list_local_devices())
```

---

#### Running Jupyter notebooks in your local computer

1. generate config

	```bash
	jupyter notebook --generate-config
	```

2. add following code into `xxx/.jupyter/jupyter_notebook_config.py`

	```bash
	c = get_config()
	c.IPKernelApp.pylab = 'inline'
	c.NotebookApp.open_browser = False
	c.NotebookApp.token = ''
	```
3. run `jupyter notebook`

	```bash
	## remote
	
	jupyter notebook
	```

4. port fowarding in local

	```bash
	## local
	
	ssh -N -f -i ~/.ssh/google_compute_engine -L 8898:localhost:8888 davidtseng@<IP-address-of-your-GPU-instance>
	```
	
	then type `localhost:8898` in your browser.
	

