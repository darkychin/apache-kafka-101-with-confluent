# Python

## Setting up env

1. Creating environment and the directory it works in

```sh
python3 -m venv env
```

`python3`: Specifies the Python 3 interpreter (as opposed to Python 2, if both are installed).
`-m venv`: Runs the built-in venv module as a script. This module is part of Python's standard library (since Python 3.3) and is used for creating isolated environments.
`env`: The name of the directory where the virtual environment will be created. You can replace "env" with any name you prefer (e.g., myproject).

Credit: GrokAI

2. Activates the environment

```sh
source env/bin/activate

or

. env/bin/activate
```

Activate the created environment "env" via its binary

Credit: GrokAI

If you facing issue with `source not found`, please check out:

- https://stackoverflow.com/questions/13702425/source-command-not-found-in-sh-shell

## Why use **config.ini** file and Python `ConfigParser` library

Read here: https://docs.python.org/3/library/configparser.html


