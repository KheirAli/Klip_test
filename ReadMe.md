# PaDIS model- KLIP method


This repository is built on top of the [PaDIS](https://github.com/jasonhu4/PaDIS) codebase.

We modify the original framework by replacing its sampling procedure with a custom sampler designed for our experiments. In addition, we include updated configuration files and experiment scripts for our task.

## Overview

Our implementation keeps the main structure of PaDIS, but introduces a new sampling strategy that better fits our setting. This repository is intended for reproducing our experiments and extending the modified pipeline.

## Main Changes Compared to PaDIS

- Replaced the original sampling routine in `PATH/TO/FILE.py` with our custom sampler.
- Added scripts for running experiments on `[dataset / task name]`.
- Updated configuration files to support the modified sampling process.
- Adjusted parts of the pipeline to match our evaluation setup.

## Installation

Clone the repository and install the required dependencies:

```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
pip install -r requirements.txt
