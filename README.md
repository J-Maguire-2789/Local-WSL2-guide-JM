# Local-WSL2-guide-JM
---------------------------------------------------------------------------------------------------------------------------
Windows subsystem for Linux 2 (WSL2) is a method to seamlessly host the Linux OS within your windows machine, this is really useful if you're a python user for bioinformatics, for me, I use python for both single-cell and spatial transcriptomics. I decided to make this guide to demonstrate how easy setting up a WSL2 environment can be, this will also cover installing mamba onto your environment, as well as an additional step for those with NVIDIA brand GPUs to utilise GPU accelerated CUDA pipelines, which are very useful for both dramatically speeding up compute times and utilising machine-learning workflows.

Author: John Maguire
---------------------------------------------------------------------------------------------------------------------------

Requirements:
Windows OS (Mac users can already natively run most linux-based pipelines)
At least 16GB RAM (WSL2 will run with half of your total RAM (e.g. 8GB/16GB RAM), leaving the other half for the windows OS to run)
Windows 10 (2004+) or 11
Virtualisation enabled in BIOS
20GB+ available space on C: drive (recommend more for your data)

Requirements for GPU acceleration:
NVIDIA brand GPUs with latest graphic drivers installed (visit https://www.nvidia.com/en-us/software/nvidia-app/ to install the NVIDIA app, which then has an option to install drivers within)
An additional 15 GB of space for GPU packages 

To check if you have a NVIDIA GPU press CTRL+Shift+Esc together to open Task Manager, then click "performance" (it looks a bit like a heartbeat) and scroll down to GPU, it should begin with NVIDIA. Some laptops have two GPUs, check both in this case. 

---------------------------------------------------------------------------------------------------------------------------

STEP 1:

Open powershell as administrator (right click windows powershell) and type "wsl --install", then restart your computer. You may get a popup at this point, this is likely the WSL2 welcome screen, you can read through this but it isn't relevant for the rest of the tutorial.

If your install fails and you get error code: 0x80370102, it means that virtualisation is likely NOT enabled in BIOS, use this code and you will find guides on how to access your BIOS to turn it on, as I will not be covering BIOS options in this guide.

STEP 2:
Often WSL will come with Ubuntu, if this is the case, skip to step 2.5.

Reopen powershell as administrator then type "wsl.exe --list --online" to get a list of all available distributions of the Linux OS, for almost all purposes Ubuntu is the best to install, now type "wsl.exe --install Ubuntu".

STEP 2.5

You'll be prompted to make a username, as a rule of thumb: Keep this lowercase, no spaces (you can use underscores as spaces though), example: yourname_local. After setting a username you'll then need to type a password, it won't appear when you type, your computer isn't broken this is something called "blank typing" and is often used for security purposes.

WSL2 will also open in C:\WINDOWS\system32, this is fine but DO NOT WORK HERE (messing with system 32 can brick your computer), immediately type "cd ~" which will bring you to your Linux home (you can write here).

STEP 3:

Type "notepad $env:USERPROFILE\.wslconfig" in a powershell window (if still in the WSL2 environment, type exit first) and then click yes when prompted, this will automatically create the .wslconfig file within your user directory on your machine, which can then be edited using one of my presets by simply pasting it in or edited manually, save the file (make sure it saves as .wslconfig and not .wslconfig.txt) and then type "wsl --shutdown" followed by "wsl ~", this restarts WSL2 to apply the changes. For those wondering, it will look something like this (don't copy this one, there are ready made configs for you in the repo):

[wsl2]

memory=8GB 
#(Half your total RAM capacity (e.g. 8GB)

swap=16GB 
#(This is sort of like a pagefile for WSL2, when you run out of usable memory during a RAM intensive process WSL2 will create a "swap file" as temporary ram on your C: drive. Swap files are incredibly slow and inefficient, but will stop an out of memory error (OOM) (unless you go over the limit you set)

[experimental]
autoMemoryReclaim=gradual 
#(This is an experimental windows feature that prevents RAM being hoarded by WSL2. Ordinarily, when you run WSL2 it will hoard all allocated ram it consumes. This setting allows WSL2 to give windows RAM it is no longer using.)

STEP 4:

Type "free -h" in your new WSL2 terminal to check your .wslconfig file saved correctly. "mem" should be the memory value you set and "swap" should be the swap value you set, it probably won't show the exact value, don't worry about that.

STEP 5:

Now we're going to make sure your WSL2 environment is up to date. With WSL2 running type "sudo apt update" first, then "sudo apt upgrade -y". You don't need to be in administrator for this step, but you will be asked for the password you set in step 2.5. 

STEP 6:

We'll now install mamba onto your WSL2 environment. Make sure you are in your Linux home directory with "cd ~", then copy this into your terminal "curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh", next type "ls -lh Miniforge3-Linux-x86_64.sh" which will bring up a readout showing the download size of the file you just downloaded, if it is around 80-130MB, then you've done it right.

STEP 7:

Type "bash Miniforge3-Linux-x86_64.sh", this will install mamba in your environment; press enter and make sure to type "yes" when prompted to accept the user agreement then press enter again to accept the default install location, type "yes" again when asked to initialise mamba. 

Now type "source ~/.bashrc", you should see (base) appear before your name.

STEP 8:

It's time to create a mamba environment, this is like a sandboxed profile for you as the user where you'll load your bioinformatic pipelines from, to begin type "mamba create -n name_your_environment_something_here python=3.12" (the name of your mamba environment follows the same rules as your username from step 2.5, as a reminder: all lowercase, no spaces (underscore is fine)).

Now type "mamba activate your_named_environment" to initialise your custom environment that you just made, for the record, every time you log on to WSL2 you will use this command to boot your environment before loading any scripts, so commit this one to memory or write it down somewhere.

STEP 9:

Okay, so now you have a functional WSL2 with a basic custom mamba environment, what now? It's time to install the packages you will be using day to day for your own analyses, type "pip install package_name_here". Here's an example of what I installed as a scRNA-seq user:

pip install scanpy

pip install numpy

pip install scipy

pip install decoupler

pip install palantir

pip install moscot

pip install pyscenic

You get the idea of the syntax, if you ever forget one, you can do this command at any time inside your mamba environment, even while scripting.

STEP 10:

You now have a couple of options for how you'd like to script. The main two I will discuss here are JupyterLab and VS Code.

JupyterLab is a web-based client for creating and organising scripts as "notebooks", which allow you to put multiple different scripts into "cells" and create long pipelines, it is very good for data analysis and exploration, but notably lacks some quality of life features which will become invaluable as you progress in your computational biology (or other field) career, namely lacking the ability to natively commit your scripts to GitHub for sharing and version control.

VS Code runs natively on your computer and interfaces directly with your WSL2 environment, rather than using a web-portal to interface with it. You can also link VS Code to your GitHub account and directly commit scripts from within the app; this is important for data reproducibility, sharing, and version control (if you make a damning mistake, which will happen, you can just roll the script back to an earlier version). I recommend VS Code, it can even run Jupyter within the app. The main drawback is it requires a bit more setup and is a little less user friendly.

JUPYTERLAB SETUP:

By far the easiest to setup, type "pip install jupyterlab", then simply type "jupyter lab --no-browser" and it will setup a browser-based interface you can access with the link provided, which looks like this "http://localhost:8888/lab?token=XXXXXXX". Copy paste that link into your preferred browser, and you're good to go.

When you want to close JupyterLab, either press "shut down" in the web browser (it's under "file") or within your terminal press CTRL+C a couple times and close the browser window.

VS CODE SETUP:

STEP 1 (VS Code): 

Type "winget install Microsoft.VisualStudioCode" into POWERSHELL as administrator, not your WSL2 environment, simplest way to do this without closing your current session is to right click the powershell icon again and select "run as administrator", which will create another powershell instance not running WSL2.

STEP 2 (VS Code):

Open VS Code on your computer and press the extensions panel (it looks like four boxes on the left side of the screen, you can also press CTRL+Shift+X at the same time to open it), now search for WSL and install the one by Microsoft. Search for Python next, and install that too.

STEP 3 (VS Code):

Restart your WSL2 environment (Remember: wsl --shutdown, then wsl ~, followed by the mamba activate command), now type "code .". This connects VS Code to your WSL2 environment.

If "code ." returns "command not found", close powershell completely and open a new one, restart your WSL2 environment and try again.

Sometimes you may need to click a button when installing extensions after connecting that says "Install in WSL:Ubuntu", you probably won't have to but just in case.

STEP 4 (VS Code):

type "pip install ipykernel" with your environment active, this is necessary to allow VS Code to use your WSL2 as the "hub" for python.

STEP 5 (VS Code):

If you'd like to use jupyter INSIDE of VS Code, go to the extensions tab again and search for "Jupyter", install the first result. Now go to file at the top and select "new file", it'll open a dropdown, select Jupyter Notebook (file extension will be file_name.ipynb).

STEP 6 (VS Code):

Whichever form of scripting you decide on using inside of VS Code, go to "Select Kernel" at the top right, you'll get a dropdown that will say "python environments", then click that and select your mamba environment.

STEP 7 (VS Code):

To check it's working properly, create a new cell with the "+" icon, then type "import sys; print (sys.executable)" and click run (the play icon). If you've set it up correctly, you should see a directory path in the output, it'll look like this:

/home/your_username/miniforge3/envs/your_mamba_environment/bin/python

if so, congratulations! you are now ready to use VS Code!

----------------------------------------------------------------------------------------------------------------------------------

GPU acceleration for NVIDIA users

STEP 1 (GPU):

Update your NVIDIA drivers using the NVIDIA app (link to install at top of readme)

STEP 2 (GPU): 

Check that WSL can see the GPU, type "nvidia-smi" in your WSL2 environment, your GPU should be listed.

Here on the top right you will also see your CUDA version, note this number down as it will determine what CUDA version you use with PyTorch in the next step.

STEP 3 (GPU):

Create a separate mamba environment exclusively for GPU acceleration, GPU packages are quite big and temperamental, so a separate environment will preserve your main packages without these packages altering their versions. Optional but highly recommended.

Type "mamba create -n your_gpu_env_name_here python=3.12" and then activate with "mamba activate your_gpu_env_name_here". Then you'll need to install pytorch, go to https://pytorch.org/ and use the listed install command as it updates frequently and I don't want to provide an outdated syntax.

On the website selector, choose Build: Stable, OS: Linux, Package: Pip, and the highest CUDA version your GPU can support, which you noted down in step 2 (do NOT pick CPU).

STEP 4 (GPU):

Inside your separate environment type:
python -c "import torch; print(torch.cuda.is_available())" 

This is to verify pytorch can see and use your GPU, it'll print "True" if it did and "False" if it cannot use your GPU. In the event of "False" this could be due to an array of issues:
1: You may have accidentally installed the CPU-only version of PyTorch, in this case go back to the website and re-select CUDA, then use that install command.
2: You have outdated graphic drivers, install the newest within the NVIDIA app.
3: Your GPU may be too old to use PyTorch.

STEP 5 (GPU):

You're now mostly ready to use GPU acceleration, well done. you'll just need to install your desired GPU packages into your GPU mamba environment. For single-cell workflows this is:

pip install scvi-tools
then going to https://rapids-singlecell.scverse.org/en/stable/installation.html to find the rapids-singlecell install command as it also updates frequently.

With rapids-singlecell I ran into some issues while installing, so I will guide you through what worked for me after troubleshooting.

STEP 1 (RAPIDS):

Type "curl -L -O https://raw.githubusercontent.com/scverse/rapids-singlecell/main/conda/rsc_rapids_26.10_cuda13.yml" (replace cuda13 with cuda12 if your nvidia-smi CUDA version is below 13). 

As this guide ages, new versions will release, simply go to this repo https://github.com/scverse/rapids-singlecell/tree/main/conda and then replace rsc_rapids_26.10_cuda13.yml with the latest version there.

STEP 2 (RAPIDS):

Type "mamba env create -f your_downloaded_file.yml", rapids works best in its own environment.

STEP 3 (RAPIDS):

type "y" when prompted and wait for the install to finish.

STEP 4 (RAPIDS):

Type "mamba env list" to see your available environments, the new rapids environment will be called "rapids_singlecell"

Activate with "mamba activate rapids_singlecell", then type the following command:
python -c "import rapids_singlecell as rsc; print(rsc.__version__)"

If this returns a version number then you're good to go.

------------------------------------------------------------------------------------------------------------------------------------

CONCLUSION

I hope this guide was helpful, if you have any questions, feel free to message me on LinkedIn (John Maguire) or open an issue on this repository and I'll try to get back to you when I can in between PhD applications and the like. Happy scripting!

------------------------------------------------------------------------------------------------------------------------------------
