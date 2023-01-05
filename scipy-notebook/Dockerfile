FROM jupyter/minimal-notebook:ubuntu-20.04
# Replace `latest` with an image tag from to ensure reproducible builds:
# https://hub.docker.com/r/jupyter/minimal-notebook/tags/
# Inspect the Dockerfile at:
# https://github.com/jupyter/docker-stacks/tree/HEAD/minimal-notebook/Dockerfile

USER root

#install Azure CLI & Azure ML Extension
RUN	wget -O- --no-verbose -q https://aka.ms/InstallAzureCLIDeb | bash 
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN az extension add -n azure-cli-ml

# add git, code formatting, bash kernel, widget suport
# switch to pip instead of conda-forge as no wheel exists for ubuntu 20.04 jupyter/minimal-notebook base image
# RUN mamba install -y jupyterlab_code_formatter jupyterlab_execute_time && \
RUN pip install --no-cache-dir \
"ipywidgets>=7.6" \
jupyterlab_code_formatter \
jupyterlab_execute_time \
jupyterlab-git \
bash_kernel \
black isort \
jupyterlab-unfold \
jupyterlab-favorites \
jupyterlab-recents && \
jupyter serverextension enable --py jupyterlab_code_formatter --sys-prefix

#Install Poetry
#USER root
RUN wget -O- --no-verbose -q https://install.python-poetry.org | POETRY_HOME=/etc/poetry python3 - --version 1.2.0
ENV PATH="${PATH}:/etc/poetry/bin"
USER ${NB_UID}

# note: when installing nb_conda_kernels we may not need to register manually with ipykernel install
# but it may also be slower or compute intense to check the available envs
# nb_conda_kernels would support showing user defined kernels without manual install step
# RUN mamba install -y nb_conda_kernels
 
# install additional package(s) as kernel...
COPY requirements.txt *.whl ./

#Shipyard VEnv
RUN mamba create -y -n shipyard.venv python=3.7
RUN mamba run  --no-capture-output -n shipyard.venv pip install ./batch_shipyard-3.9.1-py3-none-any.whl && \
    chmod 550 /opt/conda/envs/shipyard.venv/bin/shipyard && \
    ln -s /opt/conda/envs/shipyard.venv/bin/shipyard /usr/bin/ && \
    rm -f ./batch_shipyard-3.9.1-py3-none-any.whl && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

#BST Azure VEnv
RUN mamba create -y -n bst_azure.venv python=3.8
RUN mamba run --no-capture-output -n bst_azure.venv pip install ./bst_azure-0.40.42-py3-none-any.whl && \
    chmod 550 /opt/conda/envs/bst_azure.venv/bin/create_shipyard_files && \
    ln -s /opt/conda/envs/bst_azure.venv/bin/create_shipyard_files /usr/bin/ && \
    rm -f ./bst_azure-0.40.42-py3-none-any.whl && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

#Data Engineering Env
RUN mamba create -y -n data_engineering python=3.9
RUN mamba run --no-capture-output -n data_engineering pip install --no-cache-dir -r ./requirements.txt ipykernel && \
   mamba clean --all -f -y && \
   rm -rf ./requirements.txt && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

USER root
RUN mamba run --no-capture-output -n data_engineering python -m ipykernel install --prefix="/usr/local" --name data_engineering --display-name "Python (data_engineering)" && \
   python -m bash_kernel.install --prefix="/usr/local"
USER ${NB_UID}

# set the default command of the image,
# if the parent image will not launch a jupyterhub singleuser server.
# The JupyterHub "Docker stacks" do not need to be overridden.
# Set either here or in `singleuser.cmd` in your values.yaml
# CMD ["jupyterhub-singleuser"]
