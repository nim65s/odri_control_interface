odri_control_interface
----------------------

[![Test](https://github.com/open-dynamic-robot-initiative/odri_control_interface/actions/workflows/test.yml/badge.svg)](https://github.com/open-dynamic-robot-initiative/odri_control_interface/actions/workflows/test.yml)
[![Release](https://github.com/open-dynamic-robot-initiative/odri_control_interface/actions/workflows/release.yml/badge.svg)](https://github.com/open-dynamic-robot-initiative/odri_control_interface/actions/workflows/release.yml)
[![PyPI version](https://badge.fury.io/py/odri-control-interface.svg)](https://pypi.org/project/odri-control-interface)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/open-dynamic-robot-initiative/odri_control_interface/main.svg)](https://results.pre-commit.ci/latest/github/open-dynamic-robot-initiative/odri_control_interface/main)

### What it is

Common interface for controlling robots build with the odri master board.

### Installation

#### Download the package:

We use `treep` to download the required packages. Make sure your ssh key is unlocked. Then

```
mkdir -p ~/devel
pip install treep  # This installs treep
cd ~/devel
git clone git@github.com:machines-in-motion/treep_machines_in_motion.git
treep --clone master-board
treep --clone odri_control_interface
```

#### Build the package

We use [colcon](https://github.com/machines-in-motion/machines-in-motion.github.io/wiki/use_colcon)
to build this package:
```
cd mkdir -p ~/devel/workspace
colcon build
```
### Usage:

#### Demos/Examples

You find examples in forms of demos under the `demos/` folder. The demos show how to use the python and C++ interface for talking to the library and setting up a robot.

### License and Copyrights

License BSD-3-Clause
Copyright (c) 2021, New York University and Max Planck Gesellschaft.
