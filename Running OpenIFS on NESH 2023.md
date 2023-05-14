# Running OpenIFS on NESH

## Steps

1) 
1) Compile the model code
2) Make a runscript
3) Run the model

## 1) Set up your environment

You have two main directories: $HOME and $WORK. 
$HOME has very little capacity but is backed up daily. $WORK has large capacity but is not backed up. The strategy should be to keep code, e.g. model code and Jupyter notebooks, on $HOME, while model data goes on $WORK. 

Key directories:
| --------------------------- | --------------------------------------------------- | 
| `$HOME/esm/esm_tools`       | Home to ESM-Tools, where you keep runscripts        |
| `$HOME/esm/models`          | Where all model code is                             |
| `$HOME/esm/esm-experiments` | Link to $WORK/esm-experiments, where your data goes |
| --------------------------- | --------------------------------------------------- |

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

## More information
 
### Handy UNIX commands

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
| ---------------------------- | ---------------------------------------------------------- |

 
### Default configurations

A number of default configurations have been set up for this short project. You may use these as starting points and modify them as you wish. 

* Standard AMIP simulations at Tco95 and Tco199 resolutions. 
* Forecast of Desmond storm December 2015 at Tco199 resolution.
* Aquaplanet at TL255 resolution. 

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

If you wish to change the rate of entrainment in organised convection, add the following to the runscript

```bash
add_namelist_changes:
            fort.4:
                NAMCUMF:
                    ENTRORG: 1.75E-3
```

This will change the parameter when the run is being set up. 

Note that ENTRORG is defined in the NAMCUMF chapter. There are many other parameters in other chapter to tweak. For example, NAMCLDP includes a lot of cloud parameters. To change the fall speed of ice in clouds:

```bash
 add_namelist_changes:
            fort.4:
                NAMCLDP:
                    RVICE: 0.13
```

