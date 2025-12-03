{\rtf1\ansi\ansicpg1252\cocoartf2709
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\paperw11900\paperh16840\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 This repository is based on the [PaDIS](https://github.com/jasonhu4/PaDIS) codebase.  \
We replace the original sampling method with a new sampler tailored for our experiments.\
\
## Main changes compared to PaDIS\
\
- Replaced the sampling routine in `PATH/TO/FILE.py` with our custom method.\
- Added scripts for running experiments on [your dataset / task].\
- Updated configuration files to support the new sampler.\
\
## Usage\
\
```bash\
# Example:\
python run_custom_sampler.py --config configs/your_config.yaml}