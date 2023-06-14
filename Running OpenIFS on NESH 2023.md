# Running OpenIFS on NESH

## Steps

0) Pre-requisites
1) Set up environment
2) Compile the model code
3) Make a runscript
4) Run the model

## 0) Pre-requisites

Before running the model or analysing the output, you should take the time to do the following: 

1) Verify that you can log in to NESH
2) Set up password-less log in
3) Configure Jupyter notebooks 

### 1) Log in to NESH

In the terminal, type
```bash
ssh <username>@nesh-fe.rz.uni-kiel.de
```

This will ask you for your password and then (hopefully) log you in. 
Note: Log in to NESH is ONLY possible from computers from GEOMAR or CAU. 

### 2) Set up password-less log in

In the terminal, generate an SSH key: 
```bash
ssh-keygen -t rsa -b 4096 -f $HOME/.ssh/id_rsa_nopwd
```

and just press enter when prompted for password. 
This will create an SSH key on your computer in the hidden folder `.ssh/`

Verify that both the key and the public key exist: 
```bash
% ls ~/.ssh/
id_rsa_nopwd	id_rsa_nopwd.pub
```

Open `id_rsa_nopwd.pub`, and copy its contents. 

Next, log in to NESH (see step 1 above), and paste the key from before into `~/.ssh/authorized_keys` (create this file if it does not already exist). 

Now your key exists on both your local computer and NESH, so NESH will let you in without a password. 
However, we need to tell your local computer to use the key to log in. 

On your local computer: Create (if it does not exist) or modify your `.ssh/config` file. 
It should contain:

```bash
Host nesh1
   Hostname nesh-fe1.rz.uni-kiel.de
   User <your username>
   IdentityFile ~/.ssh/id_rsa
   ForwardX11 yes
```

Now, log in to NESH again with 
```bash
ssh nesh1
```

It should work without ever asking for a password. 

### 3) Set up Jupyter notebooks

You'll need to be able to run Jupyter notebooks on NESH. 

Download the `jupyter_on_HPC_setup_guide`:
```bash
git clone https://git.geomar.de/python/jupyter_on_HPC_setup_guide.git 
```

enter the scripts folder
```bash
cd jupyter_on_HPC_setup_guide/scripts/
```

and install a miniconda environment on your NESH account
```bash
./remote_jupyter_manager.sh nesh1 install
```

(this will take a while). 

Now you can start Jupyter on NESH

```bash
./remote_jupyter_manager.sh nesh1 start
```

## 1) Set up your environment

You have two main directories: $HOME and $WORK. 
$HOME has very little capacity but is backed up daily. $WORK has large capacity but is not backed up. The strategy should be to keep code, e.g. model code and Jupyter notebooks, on $HOME, while model data goes on $WORK. 

Key directories:
| Directory                   | Description                                         |
| --------------------------- | --------------------------------------------------- | 
| `$HOME/esm/esm_tools`       | Home to ESM-Tools, where you keep runscripts        |
| `$HOME/esm/models`          | Where all model code is                             |
| `$HOME/esm/esm-experiments` | Link to $WORK/esm-experiments, where your data goes |

## 2) Compile model code

Go to the `$HOME/esm/models/` directory. Here you find `oifs-43r3-v2` which is the OpenIFS model. 
You can compile the model with 
```bash
esm_master comp-oifs-43r3-v2
```

This will take some 10 minutes as it needs to compile three separate components. 
When you see something similar to
```bash
[info] compile   targets: modified=2530, unchanged=0, failed=0, total-time=1912.8s
[info] compile+  targets: modified=802, unchanged=0, failed=0, total-time=0.6s
[info] ext-iface targets: modified=1241, unchanged=0, failed=0, total-time=23.2s
[info] install   targets: modified=224, unchanged=0, failed=0, total-time=0.3s
[info] link      targets: modified=1, unchanged=0, failed=0, total-time=18.3s
[info] TOTAL     targets: modified=4798, unchanged=0, failed=0, elapsed-time=278.1s
[done] make oifs           # 286.3s
[done] make                # 286.6s
```

you know it's done. 
Now the model binaries are in `oifs-43r3-v2/bin`, and ESM-Tools will know to look for them there. 

## 3) Make a runscript

Go to `$HOME/esm/esm_tools/runscripts/oifs` where you will find a large number of runscripts for various experiments. 
You'll want to check out `oifs-43r3-climMEMODEL-NESH.yaml`. 
The runscript is written in yaml, which is very python-like. For example, it is sensitive to indendations etc. 
The comments in the runscript should explain what each setting does. 

You can run your first test run without changing anything here. 
```bash
esm_runscripts oifs-43r3-climMEMODEL-NESH.yaml -e OIFS-KSM001 
```

The option `-e` specified the experiment name. It is good practice to give each experiment a unique name. You can do this however you wish, but a common method is to name it after your location, your initials, and then a number. `OIFS-KSM001` means "Kiel, Sara Mustermann, experiment 1". 

If you see a line similar to 
```bash

```
then the job has been submitted to the queue and you may check the queue to see that it is running. 

## 4) Analyse the experiment

When your experiment has finished, you may plot the results using Jupyter notebooks on NESH. 

On your local machine, install the remote Jupyter package
```bash
```

The notebook may be found at 
```bash

```

The notebook contains some examples for plotting a few key variables, but you should definitely modify it to plot variables you are interested in and also compare simulations. 

## 5) Run more experiments
 
Now it is up to you to formulate an idea for a project, make the necessary changes to the runscript, run the experiment(s), and analyse the results.  
See "More information" below for some things that can be done and how to do it. 

## More information
 
### Handy UNIX commands

| Command                      | Result                                                     |
| ---------------------------- | ---------------------------------------------------------- |
| ls                           | List contents of directory                                 |
| vi <file>                    | Open and edit a text file                                  | 
| cd <dir>                     | Change to directory <dir>                                  |
| ls -lt                       | List contents with most recently changed files on top      |
| ls -lt | head --lines=30     | As above, but only show first 30                           |
| squeue -u $USER              | List all your currently running jobs                       |
| watch squeue -u $USER        | As above, but update every 2 seconds                       |
| sbatch <script.run>          | Submit a .run script to the queue                          |
| scancel <job ID>             | Stop a running job (get Job ID from squeue)                |
| ncview <file.nc>             | Open and plot contents of netCDF file                      |
 
### Default configurations

OpenIFS can be run at Tco95 L91 (100 km, 91 levels) and Tco199 L91 (50 km, 91 levels) or Tco199 L137 (50 km, 137 levels). 
All initial conditions are for 2008-01-01. 

### More ensemble members

OpenIFS is deterministic by default, i.e. two simulations with the same settings will yield exactly the same results. To run several diverging ensemble members you can 

* Perturb initial conditions
* Apply stochastic (random) perturbations during the run
* Make small adjustments to namelist parameters (e.g. cloud parameterisations)

The first can be easily done by setting

```yaml
perturb: 1
ensemble_id: 2
```

in the runscript which turns on perturbations of initial conditions, and generates ensemble member 2. 
Note: if you want another ensemble member you must set 

```bash
ensemble_id: 3
```

and so on. A script exists to automatically launch a multi-ensemble run.  

### Modify a namelist parameter

If you wish to e.g. change the rate of entrainment in convection to 1.5e-4, add the following to the runscript

```bash
add_namelist_changes:
            fort.4:
                NAMCUMF:
                    ENTRORG: 1.5E-3
```

This will change the parameter when the run is being set up. 

Note that ENTRORG is defined in the NAMCUMF chapter. There are many other parameters in other chapters to tweak. For example, NAMCLDP includes a lot of cloud parameters. To change the fall speed of ice in clouds. 

```bash
 add_namelist_changes:
            fort.4:
                NAMCLDP:
                    RVICE: 0.13
```

### Modify input data

All model input can in principle be modified, e.g. orography, SST, land-sea mask, surface properties (vegetation, albedo, roughness etc). 
You can change SSTs and sea-ice concentrations as well. 
The data is contained in GRIB files, but can be converted to netCDF, modified, and converted back to GRIB. 

### Directory structures

Your experiments will be stored in `/gxfs_work1/geomar/$USER/esm-experiments/`. 
If your experiment is `OIFS-KJK001`, then you should see something like

```bash
(base) [smomw352@nesh-fe1 esm-experiments]$ ls OIFS-KJK001
analysis  bin  config  couple  forcing  input  log  mon  outdata  restart  run_19790101-19790131  run_19790201-19790228  scripts  src  unknown  viz
```

| Directory                    | Use                                                     |
| ---------------------------- | ---------------------------------------------------------- |
| analysis                     | Empty                                 |
| bin                          | Executables, e.g. master.exe                               | 
| config                       | Namelists etc.                                  |
| couple                       | Only used for coupled climate models      |
| forcing                      | Empty                          |
| input                        | Input data, e.g. orography, grid files, gas concentrations |
| log                          | Log files from models                       |
| mon                          | Monitoring. Not used here.                           |
| outdata                      | All model output                |
| restart                      | Restart files for each run                     |
| run_19790101-19790131        | All files needed to run 1979-01-01 to 1979-01-31 |
| scripts                      | Copy of ESM Tools runscript, and script sent to queue |
| src                          | Not used |
| unknown                      | Things ESM-Tools does not know what to do with (nothing is ever deleted) |
| viz                          | Visualization (not used) |

The run directory, e.g. `run_19790101-19790131`, will have a similar structure 

```bash
(base) [smomw352@nesh-fe1 OIFS-KJK001]$ ls run_19790101-19790131/
analysis  bin  config  couple  forcing  input  log  mon  outdata  restart  scripts  src  unknown  viz  work
```

This directory contains everything needed to run this particular time period. 
Much of this are copies, and it will be deleted automatially after some time. 
When the run is done, everything is put in the main experiment directory. 

But we now have the `work` directory, which is where the model actually runs. 
This contains e.g. 

| File                | Description                                                     |
| ------------------- | ---------------------------------------------------------- |
| 95_4                | Input data for Tco95 grid                                 |
| context_ifs.xml     | Settings to control OpenIFS output |
| fort.4              | Namelist with model settings                                | 
| hostfile_srun       | File describing how machine executes oifs              |
| ICMGGECE3INIT       | Initial conditions (grid-point space, surface)      |
| ICMGGECE3INIUA      | Initial conditions (grid-point space, 3D)               |
| ICMSHECE3INIT       | Initial conditions (spherical harmonics)   |
| ICMCLECE3INIT       | SST and sea-ice data for each day                      |
| ifsdata             | Input data (greenhouse-gas conc, aerosols etc)          |
| ifs.stat            | Time for running each time step |
| ifs_xml             | XML files for controlling model output |
| iodef.xml           | XIOS settings |
| NODE.001_01         | Log file from OpenIFS (very big) |
| oifs                | OpenIFS executable |
| xios.x              | XIOS binary   |
| xios_client/server* | Log files from XIOS (one for each core)  |   


