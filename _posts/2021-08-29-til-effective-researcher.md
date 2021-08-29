---
toc: true
layout: post
description: Tips and Tricks for research work
categories: [research, devops, machine learning]
title: Tips and Tricks for research work
---

## SSH setup

Setting ssh config file (`~/.ssh/config`):

```
Host FLIP # access point server
    HostName access.engr.oregonstate.edu
    User ONIDusername
Host DGX # target server
    HostName submit-b.hpc.engr.oregonstate.edu
    User ONIDusername
    ProxyJump FLIP
```
*Context: at OSU, FLIP server is our access point server and DGX server is a cluster server that I would like to access, so in order to access the DGX server, I would need to access the FLIP server first. The `ProxyJump` configuration states that if I want to access the DGX server, I would need to access the FLIP server first. This step is needed for VS code's extension, `Remote - SSH`, to have access to the DGX server.*

## VS code to access cluster server
After completing `SSH setup`, do the following in VS code:
1. Install `Remote - SSH` VS code's extension  
2. Connect to the `DGX` server using `Remote - SSH`   
3. Open a Folder and select the folder of your project

## VS code to access remote Jupyter notebook server on the cluster server
After completing `SSH setup`, do the following:

1. Setting Port Forwarding, from a terminal:
    1. ssh to the server that you
    2. Install ngrok and follow their setup (https://ngrok.com/)
    3. Start the ngrok server: `./ngrok http PORT`, *replace PORT with any port*
    4. Copy the generated URL, for example `https://ad32ef7ad45e.ngrok.io`

2. Setting the Jupyter's Kernel, from a terminal:
    1. ssh to the same server as in step `1.1.`
    2. Access your conda/pip environment if needed
    3. Generate a Jupyter configuration file: `jupyter notebook --generate-config`
    4. Set Jupyter's configuration: `echo "c.NotebookApp.allow_remote_access = True" >> ~/.jupyter/jupyter_notebook_config.py`
    5. Set Jupyter's configuration: `echo "c.NotebookApp.allow_remote_access = True" >> ~/.jupyter/jupyter_notebook_config.py`
    6. Start Jupyter's server: `jupyter notebook --no-browser --port=PORT`, replace PORT with the same PORT used in `1.3.`
    7. Copy the URL token parameter, for example `?token=...`

3. Connecting everything together! (in VS code):
    1. CTRL+SHIFT+P then type `Jupyter: Specify local or remote ...`
    2. Select `Existing` then paste ngrok's url + token parameter, for example 
`https://ad32ef7ad45e.ngrok.io/?token=...`

4. (optional) to forward multiple ports, here is an example

```
# Create 4 tunnels, each for different ports, with only https enabled
# This way the ngrok process stays bellow the Free plan limit (4 tunnels)
# command: ngrok start --all # to start all of them
# command: ngrok start note tb # to run jupyter notebook server and tensorboard server only
# refer to this page for more info: https://ngrok.com/docs#multiple-tunnels

authtoken: ...
log: ngrok.log
tunnels:
  # to run jupyter notebook server
  note:
    addr: 8111
    proto: http
    bind_tls: true
  # to run visdom server
  visdom:
    addr: 8097
    proto: http
    bind_tls: true
  # to run tensorboard server
  tb:
    addr: 6006
    proto: http
    bind_tls: true
  # just an extra port available for any usage
  other:
    addr: 6969
    proto: http
    bind_tls: true
```

## GitHub Actions (CI) for multiple and parallel experiments on the cluster server

[TODO], [example](https://gist.github.com/sudomaze/c4200da526c8ce5c6d0534179f48335f)

## PyTorch Profiler to profile your model (example)

[TODO], [simple example](https://gist.github.com/sudomaze/8aad93386e2b3364edbf0759514af5cc)

## Tmux to open multiple windows in one terminal

Tmux is useful to have multiple windows open at the same time when ssh'ing to a machine so you won't need to open multiple terminals. To do that, use `tmux` to access tmux environment. Here is a script to automate the process of opening multiple windows by running a bash script:

```
#!/bin/bash
tmux new-session \; \
    send-keys 'ngrok --all' C-m \; \
    split-window -v \; \
    send-keys 'conda activate ENV; jupyter notebook --no-browser --port=PORT' C-m \; \
    split-window -v \; \
    send-keys 'conda activate ENV; python -m visdom.server' C-m \; \
    split-window -v \; \
    send-keys 'conda activate ENV; tensorboard --logdir=./outputs/logs' C-m \; \
    split-window -v \; \
    send-keys 'gitkraken' C-m \; \

# source: https://stackoverflow.com/a/40009032/5862635
```
For this case, I create 4 windows:
1. Start `ngrok`.
2. Activate my project's `ENV` then start jupyter notebook server on `PORT`.
3. Activate my project's `ENV` then start visdom server.
4. Activate my project's `ENV` then start tensorboard server.

## Unit Testing (PyTest)
[TODO]