# Snakemake executor plugin: slurm (modded)

## Overview

This modified variant of the Snakemake slurm executor is designed for use in parallelcluster environments.
It has the following features as compared to the default `snakemake-executor-plugin-slurm`:

- All calls to `sacct` are replaced with equivalent calls to `squeue`. This abrogates the need
  for a functional slurmdb.
- The plugin does not try to infer slurm username; rather, such is specified as part of
  cluster profile configuration.
- Warning messages that do not apply to parallelcluster slurm deployments are removed.
  With the default plugin, these warnings make up a substantial percentage of the
  entire Snakemake log.
- Per-job log files are relocated to function analogously to profile behavior
  from Snakemake version 7 and earlier.
- The plugin repository has documentation, making it much more likely to be usable.


## Installation

Since this is a homebrew build and I'm not pinning versions to pretend to match upstream's development,
installation with conda is a little weird. 

- Install [miniforge](https://github.com/conda-forge/miniforge).
- Run some version of the following command:

```bash
mamba create -n myenv -c "https://raw.githubusercontent.com/lightning-auriga/conda-builds/default/conda-builds" -c conda-forge -c bioconda --strict-channel-priority snakemake snakemake-executor-plugin-slurm
```

The strict channel priority is needed for mamba to solve for the modified version of the slurm plugin, which is
and will likely remain much earlier in version than the official plugin.


## Usage

With the aforementioned conda environment active, the executor plugin can be used with snakemake as follows:

- Create a profile for your slurm execution. Basically, this is a directory with a config.yaml file in it.
  See [these docs](https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles) for arguably helpful information.
- The config.yaml should contain minimally the following:

```yaml
executor: "slurm"
slurm-logdir: "logs/slurm"
slurm-keep-successful-logs: True
slurm-delete-logfiles-older-than: -1
slurm-init-seconds-before-status-checks: 20
slurm-status-attempts: 4
slurm-requeue: False
slurm-account: "ec2-user"

## other default settings for snakemake can be optionally included below.
```

- Invoke snakemake as:

```bash
snakemake --profile /path/to/profile/dir [...]
```



The logic for most of the above settings is available at [the upstream plugin's catalog page](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html).
Note that I have modded the final setting. It used to be `--slurm-no-account` and expect a boolean; I've changed this to `slurm-account` and it now expects a string
representing the slurm username that should be attached to your submitted jobs. For default parallelcluster builds, `ec2-user` is the correct setting here.


## Tested Platforms

I have tested the above plugin with the following system settings:

- AWS ParallelCluster 3.11.0
- Amazon Linux 2
- Snakemake 9.3.3

I have not tested this with later ParallelCluster versions due to issues with FSx unrelated to Snakemake. As ever,
there are no guarantees that this plugin will work with earlier or later versions of Snakemake.


## Development / Contributing / License

The code is open source and made available to you under the [MIT](LICENSE) license (MIT).
The license on most of this repo is held by upstream as specified in pyproject.toml.

Please post comments, concerns, suggestions, etc., as relate to my modifications of the base plugin in this fork.
Issues with the core plugin should be posted upstream. I have no relationship with the upstream developers.

### Contributor agreement (this modified repo only)

Submissions to this modified fork, such as PRs, will be understood to be under the following terms:

- You will be credited in this README or a contributors.txt file, but the license will remain as specified upstream as "Snakemake community."
- You retain the copyright to your submissions, but you license your contributions under the MIT license as described above.
- As author of these modifications, I attest to never knowingly including or using machine-learning code generators, including but not limited to GitHub Copilot, for these contributions. This refers to commits under my username (@lightning-auriga); no such attestation exists for other contributions upstream.
- You certify that the code is your own work (or is open-source code you are including in compliance with its license terms), and you certify you have not used machine-learning code generators, including but not limited to GitHub Copilot. You certify you are not aware of any patent encumbrances on the code beyond those covered by the MIT license. If you wish to contribute code based on machine-learning code generators, please contribute it as PRs to the upstream repository.
