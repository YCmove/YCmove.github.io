---
title: Setting up vscode debugger for pytest
date: 2024-09-19 22:08:01 +0200
tags:
   - vscode
layout: tag

# hidden: true
---

<!-- # Setting up vscode debugger for pytest -->

1. CMD+Shift+P Select a python interpreter 

![]({% link /assets/devnote_imgs/vscode_pyinterpreter.png %} " ")

2. `setting.json`: `python.testing.pytestEnabled: true`

![]({% link /assets/devnote_imgs/vscode_setting_pytest.png %} " ")

## Passing attributes to test function
Add a new filev `pytest.ini` in main directory.
```
[pytest]  
addopts = --dtype=float32,float64 --device=all
```

Now enjoy it!

![]({% link /assets/devnote_imgs/vscode_testing.png %} " ")

Or of course you can run test in termial

```
pytest -s tests/geometry/epipolar/test_triangulation.py::TestSomeClass::test_some_ function--dtype=float32,float64 --device=all
```