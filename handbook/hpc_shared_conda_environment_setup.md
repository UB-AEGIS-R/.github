# Setting Up and Using a Shared Conda Environment on HPC

This guide describes how to create, share, and use a Conda environment stored in a shared directory on a remote Linux HPC system. It also explains how individual users can register the shared environment as a Jupyter kernel.

Throughout this guide, replace:

```text
<shared_directory>
<environment_name>
<kernel_name>
```

with the appropriate paths and names for your project.

A shared environment will generally be located at:

```text
<shared_directory>/<environment_name>
```

The group folder on UB's Vortex HPC system is located at ```/projects/academic/glavrent```

## 1. Create the Shared Conda Environment

Only one user needs to create the environment initially.

```bash
conda create --prefix <shared_directory>/<environment_name> python=3.11 ipython
```

For example:

```bash
conda create --prefix /path/to/shared/project/my_conda_env python=3.11 ipython
```

Using `--prefix` places the environment at an explicit filesystem location instead of inside the user's default Conda environment directory.

> **Note:** If the shared environment has already been created, do not create it again. Simply activate the existing environment.

## 2. Tell Conda Where Shared Environments Are Located

If shared environments are stored under a common directory, add that directory to Conda's list of environment locations:

```bash
conda config --append envs_dirs <shared_directory>
```

For example:

```bash
conda config --append envs_dirs /path/to/shared/project
```

This allows Conda to discover environments stored in that directory.

Check the configured environment directories with:

```bash
conda config --show envs_dirs
```

List the environments Conda currently recognizes with:

```bash
conda env list
```

> Adding `envs_dirs` affects **Conda environment discovery**. It does not by itself register an environment as a Jupyter kernel.

## 3. Activate the Shared Environment

The most reliable way to activate a shared environment is using its full path:

```bash
conda activate <shared_directory>/<environment_name>
```

For example:

```bash
conda activate /path/to/shared/project/my_conda_env
```

Verify that the expected Python executable is being used:

```bash
which python
```

The result should point inside the shared environment:

```text
/path/to/shared/project/my_conda_env/bin/python
```

## 4. Install Packages

After activating the environment, packages can be installed with Conda:

```bash
conda install numpy scipy matplotlib
```

or, when appropriate, with `pip`:

```bash
python -m pip install <package_name>
```

Because the environment is shared, changes to the environment affect all users who use it. Package installation and upgrades should therefore be coordinated among users.

## 5. Install Jupyter Kernel Support

The shared environment needs `ipykernel` installed before it can be used from Jupyter.

Activate the environment:

```bash
conda activate <shared_directory>/<environment_name>
```

Then install `ipykernel`:

```bash
conda install ipykernel
```

This only needs to be installed once inside the shared environment.

## 6. Register the Environment as a Jupyter Kernel

Although the Conda environment itself can be shared, Jupyter kernel registrations are generally installed separately for each user.

Each user who wants to access the shared environment from Jupyter should first activate it:

```bash
conda activate <shared_directory>/<environment_name>
```

Then register it as a user-level Jupyter kernel:

```bash
python -m ipykernel install --user \
    --name <kernel_name> \
    --display-name "<display_name>"
```

For example:

```bash
python -m ipykernel install --user \
    --name project_env \
    --display-name "Python - Project Environment"
```

The environment's Python executable can also be used directly:

```bash
<shared_directory>/<environment_name>/bin/python \
    -m ipykernel install --user \
    --name <kernel_name> \
    --display-name "<display_name>"
```

An equivalent command using IPython is:

```bash
<shared_directory>/<environment_name>/bin/ipython \
    kernel install --user \
    --name <kernel_name>
```

The `python -m ipykernel` form is generally preferable because it explicitly registers the Python interpreter associated with the environment.

## 7. Restart Jupyter

After registering the kernel, restart JupyterLab or Jupyter Notebook.

Depending on how Jupyter is being run, this may require:

1. Closing the current Jupyter session.
2. Restarting JupyterLab or Jupyter Notebook.
3. Reopening the notebook.
4. Selecting the newly registered kernel.

The kernel should appear under the display name specified during installation, for example:

```text
Python - Project Environment
```

You can list all kernels registered for your account with:

```bash
jupyter kernelspec list
```

## 8. Configure Permissions for a Shared Environment

Users who need to modify the shared environment must have appropriate filesystem permissions.

To provide group read/write access:

```bash
chmod -R g+rwX <shared_directory>/<environment_name>
```

For example:

```bash
chmod -R g+rwX /path/to/shared/project/my_conda_env
```

It may also be useful to enable group inheritance on the shared project directory:

```bash
chmod g+s <shared_directory>
```

If necessary, set the appropriate Unix group ownership:

```bash
chgrp -R <group_name> <shared_directory>/<environment_name>
```

> Permission settings should follow the policies of the specific HPC system. Shared project directories may already have appropriate group ownership and permissions configured.

## 9. Typical Workflow

Once the environment has been created and the Jupyter kernel has been registered, normal use is straightforward.

### Command Line

Activate the environment:

```bash
conda activate <shared_directory>/<environment_name>
```

Run Python:

```bash
python
```

or IPython:

```bash
ipython
```

### Jupyter

Start JupyterLab or Jupyter Notebook using the normal HPC workflow and select the registered kernel:

```text
Python - Project Environment
```

## 10. What Needs to Be Done Once vs. Per User

### Done Once for the Shared Environment

Create the environment:

```bash
conda create --prefix <shared_directory>/<environment_name> python=3.11 ipython
```

Activate it:

```bash
conda activate <shared_directory>/<environment_name>
```

Install Jupyter kernel support:

```bash
conda install ipykernel
```

Configure the appropriate group ownership and permissions as needed.

### Done by Each User

Optionally tell Conda where shared environments are stored:

```bash
conda config --append envs_dirs <shared_directory>
```

Activate the environment:

```bash
conda activate <shared_directory>/<environment_name>
```

Register the environment as a Jupyter kernel:

```bash
python -m ipykernel install --user \
    --name <kernel_name> \
    --display-name "<display_name>"
```

Then restart Jupyter and select the new kernel.

## Quick Reference

```bash
# Optional: tell Conda where shared environments are stored
conda config --append envs_dirs <shared_directory>

# Activate the shared environment
conda activate <shared_directory>/<environment_name>

# Register it with Jupyter for the current user
python -m ipykernel install --user \
    --name <kernel_name> \
    --display-name "<display_name>"

# Check available Conda environments
conda env list

# Check available Jupyter kernels
jupyter kernelspec list
```

## Acknowledgment

This guide relies in part on setup instructions prepared by Flora Xia, Ph.D. Candidate at the California Institute of Technology.

## References

- [Caltech HPC: Jupyter Notebook](https://www.hpc.caltech.edu/documentation/software-and-modules/jupyter-notebook)
- [Caltech HPC: Software and Modules](https://www.hpc.caltech.edu/documentation/software-and-modules)
- [Purdue Analysis Facility: Creating and Sharing Jupyter Kernels](https://analysis-facility.physics.purdue.edu/Purdue%20Analysis%20Facility%2047dde6dc8da14392ae1747dcce5a033e/How%20to%20create%20and%20share%20Jupyter%20kernels%204dc60cf4f09344648a22da7a96b569cd.html)
