
# Quick guide to launch the TELEMAC calculations to generate your trainng data

## The overall description of the code can be found in the README.md of the repo. This guide provides a step-by-step instruction of its usage.

## These instructions are uniquely for outputing the *.npy files to generate the training data sets for LSTM or PINN. This tool also includes the PCE branch to generate and validate the surrogate model. These functionalities will not be covered in this guide.


# Step 1:

__Clone the repo__: 

_Use the branch "develop" for now as it contains the latest updates._

# Step 2:

__Define and register the work directory__: 

_The path of the work directory should be registered in the parameter "work_dir" in the YAML file "config.yml"._

# Step 3:

__Set up the case directory__: 

_Copy the TELEMAC-2D case directory under the work directory_
_

# Step 4:
__Set up the case specific parameters__: 

_This includes "case_name", "mesh_file", "cas_file", "usr_file", "results_slf", "lbf" in the YAML file._

Remark: Parameters except "**usr_file**" and "**lbf**" are obligatory. The latter two files are used only when your uncertain input parameters are defined in these files. 

# Step 5:

__Set up the template files for the TELEMAC-2D case__

_The template files include:_
* _The cas file of the case (obligatory)._
* _The usr file(s) of the case (optional)._
* _The liquid boundary files (optional)._

The template files should be renamed as a prefix "template_" followed by the name defined in the YAML paramters "cas_file", "usr_file" or "lbf_file"._

_The template files should be placed in a separate directory called "templates/" under the work directory._

# Step 6:
__Set up the tokens for the uncertain input parameters__: 

_The "token" is a string marked in the template files so that it can be replaced by a valid value._

_The token can be defined in any form. The thumb of rule is to stay distinct with the available codes in the file so that the replace operation will not accidently modify stuffs that shouldn't be modified._

_We now use the format of "@XXX" that is the symbol "@" followed by a string._

_The tokens should be marked in the template files as defined in Step 5._

_Then they should be defined in the YAML file as well in the parameters "token_cas" and "token_usr". "tokens_cas" contains the tokens used in the cas file, and "tokens_usr" is for tokens in the usr file._

# Step 7:

__Set up the uncertain parameters__

_First confirm the locations of the tokens corresponding to the . If it is in the cas file, you should activate it through setting the parameter "cas_mod" in the YAML file to 1. Same for "usr_mod". Liquid boundary files do not accept tokens. If the liquid boundary file is to be modified, set "lbf_mod" to 1._ 

Secondly, set the number of uncertain parameters and specify them in the parameter "in_dim" in the YAML file._

_Thirdly, define the distribution of each uncertain parameter in the YAML parameter "in_dist_X" where X corresponds to the index of the uncertain paramters. Currently the code only supports Uniform (in_dist_X = 0) and Normal (in_dist_X = 1). For testing purpose, Uniform is enough.

_**Remark regarding the index: the index is defined by the order of tokens defined in "token_cas" and "token_usr". The parameters listed in "token_usr" are indexed in front of the ones listed in "token_cas". For example, if "token_cas" has three listed parameters and "token_usr" has two. index 0 and 1 corresponds to the first and second parameters in "token_usr" and index 2 - 4 corresponds to the 1st, 2nd and 3rd parameter defined in "token_cas".**_

_Then, define the configurations of each distribution in the YAML parameters "ranX" with X the index. In case of Uniform, the "ranX" should be defined with the upper and lower bounds of the Uniform distribution. The meaning in "ranX" changes with different distributions._

# Step 8:

__Define the total number of training data points__

_This corresponds to the number of telemac calculations needed for the surrogate model's training set. This is configured in the YAML parameter "DoE_size"._

# Step 9:

__Define the total number of validation data points__

_This corresponds to the number of telemac calculations needed for the surrogate model's validation set. The validation set is currently automatically set as i.i.d to the training set (Tested in Uniform distribution). This is configured in the YAML parameter "validate_size"._

# Step 10:

__Define the outputs__

_This corresponds to output fields of the TELEMAC2D model that you wish to reproduce. The strings are set in the YAML parameter "out_var_list"._

# Step 11:

__Define the total number of time steps available in the TELEMAC result file__

_This is set in the YAML parameter "ntmabs". Note that the integer "ntmabs" must be equal to the division between the keyword "NUMBER OF TIME STEPS" and "GRAPHIC PRINTOUT PERIOD" in the cas file._

# Step 12:
__Set up configurations regarding slurm and telemac runtime__

This includes the following YAML parameters:
* "slurm_bs": This sets the block size for launching the TELEMAC calculations to the server. It is used to prevent overwhelming the server because the launch command of TELEMAC with slurm is very fast (in fact the launch command does not wait for the TELEMAC calculation to complete and exits immedialtely after the calculation is handled by slurm). Set this value while taking into account the total number of CPUs available and the estimated CPU time of the case you are launching.

* "otm_ncsize": This sets the number of parallel processes in the TELEMAC calculations launched by slurm. Set this value while taking into account the size of the mesh and the complexity of the model.

# Step 13:

__Set run mode for generating training and validation data set__

_You should be set right now. To launch the TELEMAC calculations to generate the training and validation sets, first set the YAML parameter "run_mode" to "0". In this case, the code will exit immediately when all training and validation calculations are launched. _

_**Remark: Different run modes are necessary since when launching the TELEMAC calculations to generate the training and validation sets, the script must be run in an interactive mode from the management node i.e. the script runs on the management node. whereas during the training and validation step, the script must be launched to the calculation nodes by slurm since it takes more resources and memories to train the surrogate.**_

# Step 14:

__Launch the script to generate training and validation data set__

_Under the working directory, use the following command_

```
  /path/to/main.py
```

_Normally, the script will start launching the calculations according to your setups. The calculations will be launched by blocks as defined in the YAML parameter "slurm_bs". At the end of each block, a 60-second sleep is implemented to give time for the server to treat the launched calculations. This duration is currently hardcoded and cannot be changed in the YAML (a TODO though). _

_The script will first launch the caluclation for the training set, then the ones for the validation set. The prefix of the calculation directories are different to distinct between these two. After all the calculations needed to generate the training and validation dataset are completed, the script will exit automatically._

# Step 15:

__Set run mode for collecting training data set__


_Once all of the calculations are fininsh, confirm that the calculations are completed through the '''squeue''' command of slurm._

_Then change the YAML parameter "run_mode" to "1", which sets the scrip to collect the results of the launche telemac caluclations and integrate them to numpy arrays._


# Step 16:

__Launch the script again to collect the training datasets.__


_Use the following command:_

```
  srun --pty --nodes=1 --ntasks-per-node=1 --cpus-per-task=8 --time=100:00:00 --partition=fat --job-name=tst-stdy /path/to/main.py
```

_The "cpus-per-task" parameter can be adjusted, but it does not really affect the performance too much unless you have parallel optimizations._

# Final:

At this step you should have the *.npy files available under your work_dir.

