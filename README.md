# Pytest and implicit namespace packages

## Implicit namespace packages (PEP 420)

The default way of defining namespace packages in python

https://packaging.python.org/en/latest/guides/packaging-namespace-packages/#native-namespace-packages

```
.
├── pyproject.toml
├── setup.cfg
└── src
    ├── mynamespace
    │   └── myproject
    │       ├── __init__.py
    │       └── tests
    │           ├── __init__.py
    │           └── test_myproject_ns.py
    └── myproject
        ├── __init__.py
        └── tests
            ├── __init__.py
            └── test_myproject.py
```

The package `mynamespace` is a namespace package since the directory `./src/mynamespace/myproject`
does NOT have a `__init__.py` file and `setup.cfg` has

```
[options]
package_dir=
	=src
packages=find_namespace:
```

## Pytest cannot handle this by default

This fails

```bash
pytest

====================================== ERRORS ======================================
______________ ERROR collecting src/myproject/tests/test_myproject.py ______________
ImportError while importing test module '/home/denolf/projects/pytest_pep420/src/myproject/tests/test_myproject.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
../../.pyenv/versions/3.11.8/lib/python3.11/importlib/__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
E   ModuleNotFoundError: No module named 'myproject.tests.test_myproject'
============================= short test summary info ==============================
ERROR src/myproject/tests/test_myproject.py
!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!
================================= 1 error in 0.06s =================================
```

Using the `importlib` import mode solves the issue but has limitations (test modules
are non-importable by each other, utility modules in the tests directories are not
automatically importable)

```bash
pytest --import-mode=importlib

src/mynamespace/myproject/tests/test_myproject_ns.py .                    [ 50%]
src/myproject/tests/test_myproject.py .                                   [100%]

=============================== 2 passed in 0.01s ===============================
```

## consider_namespace_packages

Adding this option to `pyproject.toml` solves the issue:

https://docs.pytest.org/en/stable/reference/reference.html#confval-consider_namespace_packages


```bash
(pybox_9S5crM) :~/tmp/pytest_pep420$ pytest -x
======================================== test session starts =========================================
platform linux -- Python 3.11.8, pytest-8.1.1, pluggy-1.4.0
rootdir: /home/denolf/tmp/pytest_pep420
configfile: pyproject.toml
collected 2 items                                                                                    

src/mynamespace/myproject/tests/test_myproject_ns.py .                                         [ 50%]
src/myproject/tests/test_myproject.py .                                                        [100%]

========================================= 2 passed in 0.01s ==========================================
```