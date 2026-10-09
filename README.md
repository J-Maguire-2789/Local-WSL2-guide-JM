# Local-WSL2-guide-JM
---------------------------------------------------------------------------------------------------------------------------
Windows subsystem for Linux 2 (WSL2) is a method to seamlessly host the Linux OS within your windows machine, this is really useful if you're a python user for bioinformatics, for me, I use python for both single-cell and spatial transcriptomics. I decided to make this guide to demonstrate how easy setting up a WSL2 environment can be, this will also cover installing mamba onto your environment, as well as an additional step for those with NVIDIA brand GPUs to utilise GPU accelerated CUDA pipeline which are very useful for both dramatically speeding up compute times and utilising machine-learning workflows.

Author: John Maguire
---------------------------------------------------------------------------------------------------------------------------

Requirements:
Windows OS (Mac users can already natively run most linux-based pipelines)
at least 16GB RAM (WSL2 will run with half of your total RAM (e.g. 8GB/16GB RAM), leaving the other half for the windows OS to run)
Windows 10 or later
Virtualisation enabled in BIOS
available space on C: drive

Requirements for GPU acceleration:
NVIDIA brand GPUs with latest graphic drivers installed (visit https://www.nvidia.com/en-us/software/nvidia-app/ to install the NVIDIA app, which then has an option to install drivers within)

---------------------------------------------------------------------------------------------------------------------------

STEP 1:

Open powershell as administrator (right click windows powershell) and type "wsl --install", then restart your computer.

STEP 2:
Often WSL will come with Ubuntu, if this is the case, skip to step 2.5.

Reopen powershell as administrator then type "wsl.exe --list --online" to get a list of all available distributions of the Linux OS, for almost all purposes Ubuntu is the best to install, now type "wsl.exe --install Ubuntu".

STEP 2.5

You'll be prompted to make a username, as a rule of thumb: Keep this lowercase, no spaces (you can use underscores as spaces though), example: yourname_local. After setting a username you'll then need to type a password, it won't appear when you type, your computer isn't broken this is something called "blank typing" and is often used for security purposes.

WSL2 will also open in C:\WINDOWS\system32, this is fine but DO NOT WORK HERE (messing with system 32 can brick your computer), immediately type "cd ~" which will bring you to your Linux home (you can write here).

STEP 3:

Type "notepad $env:USERPROFILE\.wslconfig" and then type "yes" when prompted, this will automatically create the .wslconfig file within your user directory on your machine, which can then be edited using one of my presets or edited manually, save the file and then type "wsl --shutdown" followed by "wsl ~", this restarts WSL2 to apply the changes. For those wondering, it will look something like this:

[wsl2]
memory=8GB #(Half your total RAM capacity (e.g. 8GB)
swap=16GB #(This is sort of like a pagefile for WSL2, when you run out of usable memory during a RAM intensive process WSL2 will create a "swap file" as temporary ram on your C: drive. Swap files are incredibly slow and inefficient, but will stop an out of memory error (OOM) (unless you go over the limit you set)

[experimental]
autoMemoryReclaim=gradual #(This is an experimental windows feature that prevents RAM being hoarded by WSL2. Ordinarily, when you run WSL2 it will hoard all allocated ram it consumes. This setting allows WSL2 to give windows RAM it is no longer using.)

STEP 4:

Type "free -h" in your new WSL2 terminal to check your .wslconfig file saved correctly. "mem" should be the memory value you set and "swap" should be the swap value you set, it probably won't show the exact value, don't worry about that.

STEP 5:

Now we're going to make sure your WSL2 environment is up to date, you MUST be running powershell as administrator for this step. with WSL2 running type "sudo apt update" first, then "sudo apt upgrade -y". you may get a popup at this point, this is likely the WSL2 welcome screen, you can read through this but it isn't relevant for the rest of the tutorial.

STEP 6:

We'll now install mamba onto your WSL2 environment. Make sure you are in your Linux home directory with "cd ~", then copy this into your terminal "curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh", next type "ls -lh Miniforge3-Linux-x86_64.sh" which will bring up a readout showing the download size of the file you just downloaded, if it is around 80-110MB, then you've done it right.

STEP 7:

Type "bash Miniforge3-Linux-x86_64.sh", this will run mamba in your environment; press enter and make sure to type "yes" when prompted to accept the user agreement then press enter again to accept the default install location, type "yes" again when asked to initialise mamba. 

Now type "source ~/.bashrc", you should see (base) appear before your name.

STEP 8:

It's time to create a mamba environment, this is like a sandboxed profile for you as the user where you'll load your bioinformatic pipelines from, to begin type "mamba create -n name_your_environment_something_here python=3.12" (the name of your mamba environment follows the same rules as your username from step 2.5, as a reminder: all lowercase, no spaces (underscore is fine)).

Now type "mamba activate your_named_environment" to initialise your custom environment that you just made, for the record, every time you log on to WSL2 you will use this command to boot your environment before loading any scripts, so commit this one to memory or write it down somewhere.

STEP 9:

Okay, so now you have a functional WSL2 with a basic custom mamba environment, what now? It's time to install the packages you will be using day to day for your own analyses, type "mamba install package_name_here". Here's an example of what I installed as a scRNA-seq user:

mamba install scanpy
mamba install numpy
mamba install scipy
mamba install decoupler
mamba install moscot
mamba install pyscenic

You get the idea of the syntax, if you ever forget one, you can do this command at any time, even while scripting.

STEP 10:

You now have a couple of options for how you'd like to script. The main two I will discuss here are JupyterLab and VS Code.

JupyterLab is a web-based client for creating and organising scripts as "notebooks", which allow you to put multiple different scripts into "cells" and create long pipelines, it is very good for data analysis and exploration, but notably lacks some quality of life features which will become invaluable as you progress in your computational biology (or other field) career, namely lacking the ability to natively commit your scripts to GitHub for sharing and version control.

VS Code runs natively on your computer and interfaces directly with your WSL2 environment, rather than using a web-portal to interface with it. You can also link VS to your GitHub account and directly commit scripts from within the app; this is important for data reproducibility, sharing, and version control (if you make a damning mistake, which will happen, you can just roll the script back to an earlier version). I recommend VS Code, it can even run Jupyter within the app. The main drawback is it requires a bit more setup and is a little less user friendly.

JUPYTERLAB SETUP:

By far the easiest to setup, type "mamba install jupyterlab", then simply type "jupyter lab --no-browser" and it will setup a browser-based interface you can access with the link provided, which looks like this "http://localhost:8888/lab?token=XXXXXXX". Copy paste that link into your preferred browser, and you're good to go.

VS CODE SETUP:

STEP 1 (VS Code): 

Type "winget install Microsoft.VisualStudioCode" into POWERSHELL as administrator, not your WSL2 environment, simplest way to do this without closing your current session is to right click the powershell icon again and select "run as administrator", which will create another powershell instance not running WSL2.

STEP 2 (VS Code):

Open VS code on your computer and press the extensions panel (it looks like four boxes on the left side of the screen, you can also press CTRL+Shift+X at the same time to open it), now search for WSL and install the one by Microsoft. Search for Python next, and install that too. 

STEP 3 (VS Code):

Restart your WSL2 environment (Remember: wsl --shutdown, then wsl ~, followed by the mamba activate command), now type "code .". This connects VS code to your WSL2 environment.

STEP 4 (VS Code):

type "mamba install ipykernel", this is necessary to allow VS code to use your WSL2 as the "hub" for python.

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

STEP 3 (GPU):

Create a separate mamba environment exclusively for GPU acceleration, GPU packages are quite big and temperamental, so a separate environment will preserve your main packages without these packages altering their verdions. Optional but highly recommended. type "mamba -n create your_gpu_env_name_here" and then activate with "mamba activate your_gpu_env_name_here". Then you'll need to install pytorch, go to https://pytorch.org/ and use the listed install command there, as it updates frequently and I don't want to provide an outdated syntax.

STEP 4 (GPU):

Inside your separate environment type "python -c "import torch; print(torch.cuda.is_available())" to verify pytorch installed, it'll print true if it did.

STEP 5 (GPU):

You're now mostly ready to use GPU acceleration, well done. you'll just need to install your desired GPU packages. For single-cell workflows this is:

pip install scvi-tools
then going to https://rapids-singlecell.scverse.org/en/stable/installation.html to find the rapids-singlecell install command as it also updates frequently.

------------------------------------------------------------------------------------------------------------------------------------

CONCLUSION

I hope this guide was helpful, if you have any questions, feel free to message me on LinkedIn (John Maguire) or open an issue on this repository and I'll try to get back to you when I can in between PhD applications and the like. Happy scripting!

------------------------------------------------------------------------------------------------------------------------------------
