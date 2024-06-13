# Python Initialization Configuration

> Added in version 3.8.

- Python can be initialized with `Py_InitializeFromConfig()` and the `PyConfig` structure.

- It can be preinitialized with `Py_PreInitialize()` and the `PyPreConfig` structure.

- There are two kinds of configuration:

    - The Python Configuration can be used to build a customized Python which behaves as the regular Python.

    - The Isolated Configuration can be used to embed Python into an application.

        - It isolates Python from the system.

## PyWideStringList

### `type PyWideStringList`

-  List of `wchar_t *` strings.

- If *length* is non-zero, *items* must be non-`NULL` and all strings must be non-`NULL`.

#### `PyStatus PyWideStringList_Append(PyWideStringList *list, const wchar_t *item)`

- Append *item* to *list*.

- Python must be preinitialized to call this function.

## PyConfig

### `type PyConfig`

#### `wchar_t *program_name`

- Program name used to initialize `executable` and in early error messages during Python initialization.

- Use `argv[0]` of `argv` if available and non-empty.

- Default: `NULL`.

- Part of the Python Path Configuration input.

## Initialization with PyConfig

### `PyStatus Py_InitializeFromConfig(const PyConfig *config)`

- Initialize Python from *config* configuration.

- The caller is responsible to handle exceptions using `PyStatus_Exception()` and `py_ExitStatusException()`.

- The current configuration (`PyConfig` type) is stored in `PyInterpreterState.config`.

- Note that since 3.11, many parameters are not calculated until initialization, and so values cannot be read from the configuration structure.

- Any values set before initialize is called will be left unchanged by initialization:

    ```c
    PyStatus init_python(const char *program_name)
    {
        PyStatus status;

        PyConfig config;
        PyConfig_InitPythonConfig(&config);

        /* Set the program name before reading the configuration
         * (decode byte string from the locale encoding).

         * Implicitly preinitialize Python. */
        status = PyConfig_SetBytesString(&config, &config.program_name,
                program_name);
        if (PyStatus_Exception(status)) {
            goto done;
        }

        /* Read all configuration at once */
        status = PyConfig_Read(&config);
        if (PyStatus_Exception(status)) {
            goto done;
        }

        /* Specify sys.path explicitly */
        /* If you want to modify the default set of paths, finish initialization
         * first and then use PySys_GetObject("path") */
        config.module_search_paths_set = 1;
        status = PyWideStringList_Append(&config.module_search_paths,
                L"/path/to/stdlib");
        if (PyStatus_Exception(status)) {
            goto done;
        }
        status = PyWideStringList_Append(&config.module_search_paths,
                L"/path/to/more/modules");
        if (PyStatus_Exception(status)) {
            goto done;
        }

        status = PyConfig_SetString(&config, &config.executable,
                L"/path/to/my_executable");
        if (PyStatus_Exception(status)) {
            goto done;
        }

        status = Py_InitializeFromConfig(&config);

    done:
        PyConfig_Clear(&config);
        return status;
    }
    ```

> ##### Error
>
> ```c
> PyStatus init_python(const char *program_name)
> {
> 	PyStatus status;
> 
> 	PyConfig config;
> 	PyConfig_InitPythonConfig(&config);
> 
> 	status = PyConfig_SetBytesString(&config, &config.program_name,
> 			program_name);
> 	if (PyStatus_Exception(status))
> 		goto done;
> 
> 	status = PyConfig_Read(&config);
> 	if (PyStatus_Exception(status))
> 		goto done;
> 
> 	config.module_search_paths_set = 1;
> 	status = PyWideStringList_Append(&config.module_search_paths, L".");
> 	if (PyStatus_Exception(status))
> 		goto done;
> 
> 	status = Py_InitializeFromConfig(&config);
>   if (PyStatus_Exception(status)) {
>       fprintf(stderr, "Error initializing from config: %s\n", status.err_msg);
>       goto done;
>   }
> 
> done:
> 	PyConfig_Clear(&config);
> 	return status;
> }
> ```
>
> ```
> Python path configuration:
>   PYTHONHOME = (not set)
>   PYTHONPATH = (not set)
>   program name = './a.out'
>   isolated = 0
>   environment = 1
>   user site = 1
>   safe_path = 0
>   import site = 1
>   is in build tree = 0
>   stdlib dir = ''
>   sys._base_executable = '<path of a.out>'
>   sys.base_prefix = '/usr'
>   sys.base_exec_prefix = '/usr'
>   sys.platlibdir = 'lib'
>   sys.executable = '<path of a.out>'
>   sys.prefix = '/usr'
>   sys.exec_prefix = '/usr'
>   sys.path = [
>     '<module search path>',
>   ]
> Error initializing from config: failed to get the Python codec of the filesystem encoding
> XXX readobject called with exception set
> Fatal Python error: error reading frozen getpath.py
> Python runtime state: core initialized
> 
> Current thread 0x00007a55f6abb740 (most recent call first):
>   <no Python frame>
> ```

## Python Path Configuration

- `PyConfig` contains multiple fields for the path configuration:

    - Path configuration inputs

    - Path configuration output fields

        - `PyConfig.module_search_paths_set`, `PyConfig.module_search_paths`

- If at least one "output field" is not set, Python calculates the path configuration to fill unset fields.

- If `module_search_paths_set` is equal to `0`, `module_search_paths` is overridden and `module_search_paths_set` is set to `1`.

- It is possible to completely ignore the function calculating the default path configuration by setting explicitly all path configuration output fields listed above.

- A string is considered as set even if it is non-empty.

- `module_search_paths` is considered as set if `module_search_paths_set` is set to `1`.

- In this case, `module_search_paths` will be used without modification.

### `PyWideStringList module_search_paths`

### `int module_search_paths_set`

- Module search paths: `sys.path`.

- If `module_search_paths_set` is equal to `0`, `Py_InitializeFromConfig()` will replace `module_search_paths` and sets `module_search_paths_set` to `1`.

- Default: empty list (`module_search_paths`) and `0` (`module_search_paths_set`).

- Part of the Python Path Configuration output.
