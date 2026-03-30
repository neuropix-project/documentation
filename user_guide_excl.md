# NEUROPix User's Guide to ExCL
This document is meant to provide a minimal, practical user guide for members of the NEUROPix project. The many features of the Experimental Computing Laboratory (ExCL) and detailed usage of software are beyond the scope of this guide but appropriate links are included in their dedicated sections.  

## Access to ExCL
### https://docs.excl.ornl.gov/

1. [Request an account](https://www.excl.ornl.gov/accessing-excl/) on ExCL for the project 'NEUROPix'
2. Once you receive an email that you now have an active ExCL account, go through the [ExCL Remote Development](https://docs.excl.ornl.gov/quick-start-guides/excl-remote-development) roadmap to set up SSH keys, Git access, VS Code, and Foxy Proxy. The rest of the guide will assume these are already set up. 
3. [Access the login node](https://docs.excl.ornl.gov/excl-support/access)
    - Assumed strategy in this guide: ssh key generated on local computer and shared on ExCL account in `ssh/authorized_keys`. It’s nice to add the key to an ssh agent for easier login. 
    - It’s also nice to define the hosts in a local `.ssh/config` file which may look like the following:
        ```config
        Host excl
            HostName login.excl.ornl.gov
            AddKeysToAgent yes
            UseKeychain yes
            IdentityFile ~/.ssh/excl_id_rsa
            DynamicForward 9090

        Host * 
            User i8x
            ForwardAgent yes
            ForwardX11 yes
        ```
    - Access the login node in terminal like 
        ```bash
        ssh excl
        ```
        - If the ForwardX11 line is not included in your Host excl config, then enable X11 forwarding like `ssh –Y excl`
    - Access a specific node in terminal like
        ```bash
        ssh -J excl NODENAME 
        ```
        - It is cleaner and easier to configure your `.ssh/config` to include the desired node. Then connection could look like `ssh NODENAME`
        - Possible NEUROPix nodes = zenith, zenith2, oswald, milan0
        - Home directory and all others are mirrored in all nodes
    - Can access ExCL through a ThinLinc UI by opening https://login.excl.ornl.gov:300/ in a web browser 
        - Then log in with UCAMS 
        - Can SSH to another node in the terminal inside the UI 
        - With Foxy Proxy, you can use `https://NODENAME.ftpn.ornl.gov:300/` to access any node with ThinLinc running. 
4. Project files (example configuration files, data, etc.) live in `/auto/projects/neuropix`. The projects folder might need automounting on the first navigation, so you may need to type out (or create a shortcut for) the full path.
5. Tools for NEUROPix work (ROOT, Geant4, Allpix-Squared) are installed in a Docker container accessed through Apptainer. The Apptainer image file for this containter is stored at `/auto/projects/neuropix/allpix-squared-gui.sif`. This image can also be pulled from [Harbor](https://savannah.ornl.gov/harbor/projects/192/repositories/allpix-squared-gui/artifacts-tab). 

## Running NEUROPix Software

1. To run shared software (ROOT, Geant4, allpix-squared) installed via Apptainer, either open the whole container and act within that space OR call an executable script that lives within the container 
    - [Allpix-Squared](https://allpix-squared.docs.cern.ch/)
        - Run a script like
            ```bash
            apptainer exec /auto/projects/neuropix/allpix-squared-gui.sif allpix -c /auto/projects/neuropix/allpix-config/neuro.conf 
            ```
            OR
            ```bash
            apptainer run /auto/projects/neuropix/allpix-squared-gui.sif  
            allpix -c /auto/projects/neuropix/allpix-config/neuro.conf
            ````
    - [ROOT](https://root.cern.ch/)
        - Run a script like
            ```bash
            apptainer exec /auto/projects/neuropix/allpix-squared-gui.sif root -l /auto/projects/neuropix/0_data_root/<FILE> 
            ```
            OR
            ```bash
            apptainer run /auto/projects/neuropix/allpix-squared-gui.sif  
            root -l /auto/projects/neuropix/0_data_root/<FILE>
            ````
    - Librarires for software in the container are in `usr/local/lib`
        - You must be in the container to see them
        - Access the container first like `apptainer run /auto/projects/neuropix/allpix-squared-gui.sif`
2. Git and Python should already be present in your ExCL space. 
    - Python 3.12.3 in Apptainer 
3. Custom analysis scripts, Git resources, etc. may need additional Python libraries. To create a custom Python environment, use `uv`  
    - Replaces pip, pyenv, virtualenv, and more - https://docs.astral.sh/uv/  
    - Install to onto any system in your space (will propagate to other nodes and into container) 
        ```bash
        curl -LsSf https://astral.sh/uv/install.sh | sh 
        ```
    - If `uv` does not propagate into the container, add the following to `/home/<UCAMS>/.bashrc` (create the file if it doesn’t exist )
        ``` bash
        . "$HOME/.local/bin/env" 
        ```
4. Example usage
    - Create a virtual environment (https://docs.astral.sh/uv/pip/environments/)  
        ```bash
        uv venv neuropix --seed 
        ```
        - This should have created a folder with the same name as the virtual environment (the last argument to the above command)
        - You can also navigate to the project folder and run `uv venv` to use the default `.venv` path 
        - The `--seed` argument  installs pip in the virtual environment
    - To activate the environment, run  
        ```bash
        source .neuropix/bin/activate 
        ```
    - Activate the environment and install relevant python packages. For example, to install numpy activate the environment and run 
        ```bash
        uv pip install numpy 
        ```
    - Deactivate the environment when you’re finished like 
        ```bash
        deactivate 
        ```
    - All this work can be done outside the Apptainer container. 
    - To run a code which imports ROOT (which only exists in the apptainer), run by opening the Apptainer like
        ```bash
        apptainer run  /auto/projects/neuropix/allpix-squared-gui.sif 
        ```
    - Once in the Apptainer, activate the virtual environment like 
        ```bash
        source .neuropix/bin/activate
        ```
    - Then, still in the Apptainer, run the code like normal 
        ```bash
        python CODENAME.py
        ``` 
 
