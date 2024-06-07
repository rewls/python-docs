# `venv` -- Creation of virtual environments

> Python version 3.12.3

> Added in version 3.3.

- Source code: Lib/venv

---

- The `venv` module supports creating lightweight "virtual environments", each with their own independent set of Python packages installed in their site directories.

- A virtual environment is created on top of an existing Python installation, known as the virtual environment's "base" Python, and may optionally be isolated from the packages in the base environment, so only those explicitly installed in the virtual environment are available.

- When used from within a virtual environment, common installation tools such as pip will install Python packages into a virtual environment without needing to be told to do so explicitly.

- A virtual environment is (amongst other things):

    - Used to contain a specific Python interpreter and software libraries and binaries which are needed to support a project (library or application).

    - Contained in a directory, conventionally either named `venv` or `.venv` in the project directory, or under a container directory for lots of virtual environments, such as `~/.virtualenvs`.

    - Not checked into source control systems such as Git.

    - Considered as disposable -- it should be simple to delete and recreate it from scratch.

        - You don’t place any project code in the environment

    - Not considered as movable or copyable – you just recreate the same environment in the target location.

- See PEP 405 for more background on Python virtual environments.

> See also: Python Packaging User Guide: Creating and using virtual environments

## Creating virtual environments

- Creation of virtual environments is done by executing the command `venv`:

    ```shell
    $ python -m venv /path/to/new/virtual/environment
    ```

- Running this command creates the target directory (creating any parent directories that don’t exist already) and places a `pyvenv.cfg` file in it with a home key pointing to the Python installation from which the command was run.

- It also creates a `bin` subdirectory containing a copy/symlink of the Python binary/binaries (as appropriate for the platform or arguments used at environment creation time).

- It also creates an (initially empty) `lib/pythonX.Y/site-packages` subdirectory.

- If an existing directory is specified, it will be re-used.

> Changed in version 3.5: The use of `venv` is now recommended for creating virtual environments.

## How venvs work

- When a Python interpreter is running from a virtual environment, `sys.prefix` and `sys.exec_prefix` point to the directories of the virtual environment, whereas `sys.base_prefix` and `sys.base_exec_prefix` point to those of the base Python used to create the environment.

- It is sufficient to check `sys.prefix != sys.base_prefix` to determine if the current interpreter is running from a virtual environment.

- This will prepend that directory to your `PATH`, so that running python will invoke the environment’s Python interpreter and you can run installed scripts without having to use their full path.

|Platform|Shell|Command to activate virtual environment|
|-|-|-|
|POSIX|bash/zsh|`$ source <venv>/bin/activate`|

- You don’t specifically need to activate a virtual environment, as you can just specify the full path to that environment’s Python interpreter when invoking Python.

- In order to achieve this, scripts installed into virtual environments have a “shebang” line which points to the environment’s Python interpreter, i.e. `#!/<path-to-venv>/bin/python`.

- This means that the script will run with that interpreter regardless of the value of `PATH`. 

- When a virtual environment has been activated, the `VIRTUAL_ENV` environment variable is set to the path of the environment.

> ##### Warning
>
> - Because scripts installed in environments should not expect the environment to be activated, their shebang lines contain the absolute paths to their environment’s interpreters.
>
> - Because of this, environments are inherently non-portable, in the general case.
>
> - You should always have a simple means of recreating an environment (for example, if you have a requirements file requirements.txt, you can invoke pip install -r requirements.txt using the environment’s pip to install all of the packages needed by the environment).

- You can deactivate a virtual environment by typing `deactivate` in your shell.
