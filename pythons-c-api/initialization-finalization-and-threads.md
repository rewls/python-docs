# Initialization, Finalization, and Threads

## Before Python Initialization

- In an application embedding Python, the `Py_Initialize()` function must be called beforeusing any other Python/C API functions; with the exception of a few functions and the global configuration variables.

## Initializing and finalizing the interpreter

### `void Py_Initialize()`

- Part of the Stable ABI.

- Initialize the Python interpreter.

- The initializes the table of loaded modules (`sys.modules`), and creates the fundamental modules `builtins`, `__main__` and `sys`.

- It also initializes the module search path (`sys.path`).

- This is a no-op when called for a second time (without calling `Py_FinalizeEx()` first).

- Use the `Py_InitializeFromConfig()` function to customize the Python Initialization Configuration.

### `void Py_InitializeEx(int initsigs)`

- Part of the Stable ABI.

- This function  works like `Py_Initialize()` if *intsigs* is `1`.

- If *initsigs* is `0`, it skips initialization registration of signal handlers, which might be useful when Python is embedded.

- Use the `Py_InitializeFromConfig()` function to customize the Python Initialization Configuration.

### `int Py_IsInitialized()`

- Part of the Stable ABI.

- Return true (nonzero) when the Python interpreter has been initialized, false (zero) if not.

- After `Py_FinalizeEx()` is called, this returns false until `Py_Initialize()` is called again.

### `int Py_FinalizeEx()`

- Part of the Stable ABI since version 3.6.

- Undo all initializations made by `Py_Initialize()` and subsequent use of Python/C API functions, and destroy all sub-interpreters (see `Py_NewInterpreter()` below) that were created and not yet destroyed since the last call to `Py_Initialize()`.

- Ideally, this frees all memory allocated by the Python interpreter.

- This is no-op when called for a second time  (without calling `Py_Initialize()` again first).

- Normally the return value is `0`.

- If there were errors during finalization (flushing buffered data), `-1` is returned.

- This function is provided for a number of reasons.

    - An embedding application might want to restart Python without having to restart the application itself.

    - During a hunt for memory leaks in an application a developer might want to free all memory allocated by Python before exiting from the application.

- Bugs and caveats:

    - The destruction of modeuls and objects in modules is done in random order; this may cause desctructors (`__del__()` methods) to fail when they depend on other objects or modules.

    - Small amounts of memory allocated by the Python interpreter may not be freed (if you fin a leak, please report it).

    - Memory tied up in circular references between objects is not freed.

- Some memory allocated by extension modules may not be freed.

- Some extensions may not work properly if their initialization routine is called more than once; this can happen if an application calls `Py_Initialize()` and `Py_FinalizeEx()` more than once.

- Raises an auditing event `cpython._PySys_ClearAuditHooks` with no arguments.

> Added in version 3.6.
