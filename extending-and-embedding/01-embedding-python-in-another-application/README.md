# 1. Embedding Python in Another Application

> - Python version 3.12.3
>
> - Environment: Arch Linux

- Enrich your C/C++ application by embedding Python in it.

- If you embed Python, the main program may have nothing to do with Python -- instead, some parts of the application occasionally call the Python interpreter to run some Python code.

- There are several different ways to call the interpreter: you can pass a string containing Python statements to `PyRun_SimpleString()`, or you can pass a stdio file pointer and a file name (for identification in error messages only) to `PyRun_SimpleFile()`.

- You can also call the lower-level operations described in the previous chapters to construct and use Python objects.

## 1.1 Very High Level Embedding

- This interface is intended to execute a Python script without needing to interact with the application directly.

> ##### [time.c](time.c)

- The `Py_SetProgramName()` function should be called before `Py_Initialize()` to inform the interpreter about paths to Python run-time libraries.

- Next, the `Py_Initialize()`, followed by the execution of a hard-coded Python script that prints the date and time.

- Afterwards, the `Py_FinalizeEx()` call shuts the interpreter down, follwed by the end of the program.

- Getting the Python code from a file can better be done by using the `PyRun_SimpleFile()` function, which saves you the trouble of allocating memory space and loading the file contents.

## 1.2. Beyond Very High Level Embedding: An overview

- The high level interface gives you the ability to execute arbitrary pieces of Python code from your application, but exchanging data values is quite cumbersome to say the least.

- If you want that, you should use lower level calls. At the cost of having to write more C code, you can achieve almost anything.

- It should be noted that extending Python and embedding Python is quite the same activity, despite the different intent.

- Most topics discussed in the previous chapters are still valid.

- This chapter will not discuss how to convert data from Python to C and vice versa.

- Since these aspects do not differ from extending the interpreter, you can refer to earlier chapters for the required information.

## 1.3. Pure Embedding

- The first program aims to execute a function in a Python script.

- Like in the section about the very high level interface, the Python interpreter does not directly interact with the application (but that will change in the next section).

> ##### [call.c](call.c)

- The code loads a Python script using `argv[1]`, and calls the function named in `argv[2]`.

- Its integer arguments are the other values of the `argv` array.

> ##### [multiply.py](multiply.py)

```shell
$ ./call multiply multiply 3 2
```

- The interesting part with respect to embedding Python starts with

    ```c
    Py_Initialize();
    pName = PyUnicode_FromString(argv[1]);
    /* Error checking of pName left out */
    pModule = PyImport_Import(pName);
    ```

- After initializing the interpreter, the script is loaded using `PyImport_Import()`.

- This routine needs a Python string as its argument, which is constructed using the 

```c
pFunc = PyObject_GetAttrString(pModule,argv[2]);

if (pFunc && PyCallable_Check(pFunc)) {
    ...
}
Py_XDECREF(pFunc);
```

- Once the script is loaded, the name we're looking for is retrieve using `PyObject_GetAttrString()`.

- If the name exists, and the object returned is callable, you can safely assume that it is a function.

- The program then proceeds by constructing a tuple of argument as normal.

- The call to the Python function is then made with:

    ```c
    pValue = PyObject_CallObject(pFunc, pArgs);
    ```

- Upon return of the function, `pValue` is either `NULL` or it contains a reference to the return value of the function.

- Be sure to release the reference after examining the value.

## 1.6. Compiling and Linking under Unix-like systems

- It is not necessarily trivial to find the right flags to pass to your compiler in order to embed the Python interpreter into your application, particularly because Python needs to load library modules implemented as C dynamic extensions (`.so` files) linked against it.

- To find out the required compiler and linker flags, you can execute the `python*X*.*Y*-conifg` script which is generated as part of the installation process (a `python3-config` script may also be available).

- This script has several options, of which the following will be directly useful to you:

    - `python*X*.*Y*-config --cflags` will give you the recommended flags when compiling.

    - `python*X*.*Y*-config --ldflags --embed` will give you the recommended flags when linking

> ##### Note
>
> - To avoid confusion between several Python installations, is is recommended that you use the absolute path to `python*X*.*Y*-config`, as in the above example.
