# Nerve Management System API Python bindings

## Description

This repository implements Python binding for the Nerve Management System API. It also includes a command-line tool using these bindings to demonstrate how to use them.

## Development Environment

These scripts have been developed and tested with Python 3.12, and it is recommended to run them with Python 3.12 or later. If you run the scripts with an older version of Python, adaptations might be necessary. It is also recommended to run the scripts within a virtual environment.

> **NOTE:** Instructions below are for Linux operating systems. For information on how to create a virtual environment on Windows, please refer to [the official Python documentation](https://python.land/virtual-environments/virtualenv#How_to_create_a_Python_venv).

To create a new development environment using a virtual Python environment run the steps listed below. This example uses `n_venv` as the environment name.

1. Create the virtual Python environment.
```
python3 -m venv n_venv
```

2. Activate the environment.
```
source n_venv/bin/activate
```

3. Install dependencies.
```
pip install -r requirements.txt
```

4. Add source code directories of this repository to `PYTHONPATH`.
```
export PYTHONPATH="${PYTHONPATH}:$(pwd):$(pwd)/examples/nerve-clt"
```

5. Check if everything works as intended.
```
python examples/nerve-clt/nerve_clt --help
```

6. When you are done you can deactivate the environment.
```
deactivate
```

Steps 4 to 6 must be repeated whenever you log out or reboot your system.

## Packaging
The Nerve Management System API Python bindings can be packaged as Python wheel for easier installation. To create the **.whl* file run
```
python setup.py bdist_wheel
```
The resulting package can be found in the *dist* directory. To install the package run
```
pip install dist/nerve*.whl
```

## License

The source code is released under MIT license (see the LICENSE file).


# Example - Command-line Tool

The repository includes
 - The Python bindings for the Nerve Management System API located in directory *nerve*.
 - A command line tool as an example for how to use the Nerve Management System API Python binding. This source code can be found in *examples/nerve-clt*. The individual Python files of this example are structured along the objects they work on. To accomplish a specific task using the API functions, looking into the implementation of the corresponding command in the commands directory may be a good starting point.


## Command-line use

The command line tool can be started via
```
python examples/nerve-clt/nerve_clt
```
Use `--help` for usage details or refer to the help output below:

```
positional arguments:
  {set_login,logout,list_workloads,create_workload,create_label,get_labels,delete_label,create_wl_template,delete_workloads,list_nodes,reboot_nodes,start,stop,restart}
                        Available sub-commands:
    set_login           Checks and sets login credentials. Uses file, environment vars or prompt.
    logout              Log out and delete session data.
    list_workloads      List workloads and store them.
    create_workload     Create a new workload.
    create_label        Create a new label.
    get_labels          List the labels in the system.
    delete_label        Delete a label.
    create_wl_template  Creates a workload definition template from the first workload in the workload list.
    delete_workloads    Delete workloads from a workload list file. Use list_workloads to create a list file.
    list_nodes          List nodes and create a node list or add to the list. See the functions help for more information.
    reboot_nodes        Reboot nodes from a node list file.  See list_nodes help for details.
    start               Start workloads from a workload list file.  See list_nodes help for details.
    stop                Stop workloads from a workload list file. See list_nodes help for details.
    restart             Restart workloads from a workload list file.  See list_nodes help for details.

options:
  -h, --help            show this help message and exit
```

As stated in the help message of the command line tool, credentials may be provided in four different ways for the `set_login` command:

- via command line arguments: `python examples/nerve-clt/nerve_clt set_login -u https://my-management-system.nerve.cloud --username myusername --password mypassword`
- via environment variables (set the `NERVE_URL`, `NERVE_USERNAME`, and `NERVE_PASSWORD` environment variables). Check the *set_login_environment_vars.sh* script to understand the naming of the variables.
- via `.ini` file. The file extension will be added automatically. So when using the file `credentials.ini` type: `python examples/nerve-clt/nerve_clt set_login --file credentials` If no specific file is provided, the command line tool will create one in the current user's homer directory under *~/.nerve_clt/credentials.ini*. **NOTE:** Credentials are stored as clear text.
- via interactive keyboard input (if none of the above is provided, the script will interactively ask for the credentials): `python examples/nerve-clt/nerve_clt set_login`

A credentials file must have the following form:
```ini
[Credentials]
url = https://my-management-system.nerve.cloud
username = myusername
password = mypassword
```
When working with multiple Management Systems it might make sense to work with `.ini` files and create one file per Management System.

### Example Usage

To use the command line tool run `python examples/nerve-clt/nerve_clt`. Use `--help` to get detailed information about all available commands.

Before any of the commands can be run on a Management System, one needs to login. There are several ways to provide credentials. One way is to run the `set_login` command and provide the credentials and the Management System URL interactively:

```bash
python examples/nerve-clt/nerve_clt set_login
# provide the requested information
```

When running the command, a login will be performed and the session information will be stored in the `session_id.ini` file. To logout one can either run the `python examples/nerve-clt/nerve_clt logout` command. If you delete the `session_id.ini` file, a new login will be required.

Now we can perform operations on the Management System such as listing all the Docker workloads that are available on the Management System:

```bash
python examples/nerve-clt/nerve_clt list_workloads --type docker --output workloads.json --human
```
This will write the result into the JSON file *workloads.json* and also print it to the command line in a human-readable way. For more details about the command, check the help with `python examples/nerve-clt/nerve_clt list_workloads --help`.

Another use case might be to get a list of all nodes where a specific workload version is currently deployed:
```bash
python examples/nerve-clt/nerve_clt list_nodes -wn nginx -vn v1 --human --file nodes.json
```
This lists all nodes where the workload with the name "nginx" is deployed in version "v1" and saves the output as JSON into the *nodes.json*.

The scripts also provide a workflow to create a new workload. Simply define the workload via a JSON file. To make it easier to create such a file, a template can be created from an existing workload:

1. To create a Docker workload first fetch a JSON file that contains such workload definitions from existing workloads on the Management System.

    ```bash
    python examples/nerve-clt/nerve_clt list_workloads --type docker --output workloads.json
    ```

2. Take the first workload from that query and create a template from it for the new workload to be created.

    ```bash
    python examples/nerve-clt/nerve_clt create_wl_template --input_file workloads.json --output_file wl_def.json
    ```

3. Now open the *wl_def.json* file with a text editor and adjust it to your needs to represent the new workload to be created and save it e.g. as *new_wl.json*
4. The new workload can now be created on the Management System with the following command.

    ```bash
    python examples/nerve-clt/nerve_clt create_workload --template new_wl.json
    ```

## Use of the Nerve Management System API Python bindings

The API functions require one commonly held state which is the session ID after login. The session is stored in the current user's home directory under *~/.nerve_clt/session_id.ini*. This was done to avoid handing the state through all calls.
All API functions make extensive use of exceptions to inform the user about unforeseen problems in the call. Make sure to expect those.

## Packaging
The command line tool can be packaged as a Python wheel. To create the package run
```
cd examples/nerve-clt
python setup.py bdist_wheel
```
The **.whl* file can be found in *examples/nerve-clt/dist*. To install the package run
```
pip install examples/nerve-clt/dist/nerve_clt*.whl
```
This installs the command line tool and makes it available via command `nerve` at your command line.

## Using with Docker
A Dockerfile is available that builds an image with the command line tool example installed. It can be found at *examples/nerve-clt/Dockerfile*. To build the image run
```
docker build -f examples/nerve-clt/Dockerfile -t nerve-clt .
```
To use the Docker image run `docker run -it --rm nerve-clt` followed by the command to run, i.e. `docker run -it --rm nerve-clt --help`. It is also possible to run `docker run -it --rm --entrypoint sh nerve-clt` to create an interactive shell. The command line tool example can then be run in there via command `nerve`, i.e. `nerve --help`. To exit the interactive shell of the Docker container use `exit`.
