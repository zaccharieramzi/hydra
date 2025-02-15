---
id: intro
title: Overview
sidebar_label: Introduction
---

import GithubLink from "@site/src/components/GithubLink"

Hydra is highly configurable. Many of its aspects and subsystems can be configured, including:
* The Launcher
* The Sweeper
* Logging
* Output directory patterns
* Application help (--help and --hydra-help)

The Hydra config can be customized using the same methods you are already familiar with from the tutorial.
You can include some Hydra config snippet in your own config to override it directly, or compose in different
configurations provided by plugins or by your own code. You can also override everything in Hydra from the command 
line just like with your own configuration.

The Hydra configuration itself is composed from multiple config files. here is a partial list:
```yaml title="hydra/config"
defaults:
  - job_logging : default     # Job's logging config
  - launcher: basic           # Launcher config
  - sweeper: basic            # Sweeper config
  - output: default           # Output directory
```
You can view the Hydra config structure <GithubLink to="hydra/conf/__init__.py">here</GithubLink>.

You can view the Hydra config using `--cfg hydra`:
<details>
<summary> $ python my_app.p <b>--cfg hydra</b> (Click to expand)</summary>

```yaml
hydra:
  run:
    dir: outputs/${now:%Y-%m-%d}/${now:%H-%M-%S}
  sweep:
    dir: multirun/${now:%Y-%m-%d}/${now:%H-%M-%S}
    subdir: ${hydra.job.num}
  launcher:
    _target_: hydra._internal.core_plugins.basic_launcher.BasicLauncher
  sweeper:
    _target_: hydra._internal.core_plugins.basic_sweeper.BasicSweeper
    max_batch_size: null
  hydra_logging:
    version: 1
    formatters:
    ...
```
</details>


## Accessing the Hydra config
The Hydra config is large. To reduce clutter in your own config it's being deleted from the config object
Hydra is passing to the function annotated by `@hydra.main()`.

There are two ways to access the Hydra config:

#### In your config, using the `hydra` resolver:
```yaml
config_name: ${hydra:job.name}
```
Pay close attention to the syntax: The resolver name is `hydra`, and the `key` is passed after the colon.

#### In your code, using the HydraConfig singleton.
```python
from hydra.core.hydra_config import HydraConfig

@hydra.main()
def my_app(cfg: DictConfig) -> None:
    print(HydraConfig.get().job.name)
```

The following variables are populated at runtime.  

### hydra.job:
The **hydra.job** node is used for configuring some aspects of your job. 
Below is a short summary of the fields in **hydra.job**. 
You can find more details in the [Job Configuration](job.md) page.

Fields under **hydra.job**:
- **name** : Job name, defaults to the Python file name without the suffix. can be overridden.
- **override_dirname** : Pathname derived from the overrides for this job
- **chdir**: If `True`, Hydra calls `os.chdir(output_dir)` before calling back to the user's main function.
- **id** : Job ID in the underlying jobs system (SLURM etc)
- **num** : job serial number in sweep
- **config_name** : The name of the config used by the job (Output only)
- **env_set**: Environment variable to set for the launched job
- **env_copy**: Environment variable to copy from the launching machine
- **config**: fine-grained configuration for job

### hydra.runtime:
Fields under **hydra.runtime** are populated automatically and should not be overridden.
- **version**: Hydra's version
- **cwd**: Original working directory the app was executed from
- **output_dir**: This is the directory created by Hydra for saving logs and
  yaml config files, as configured by [customizing the working directory pattern](workdir.md).
- **choices**: A dictionary containing the final config group choices.
- **config_sources**: The final list of config sources used to compose the config.

### Resolvers provided by Hydra
Hydra provides the following [OmegaConf resolvers](https://omegaconf.readthedocs.io/en/latest/usage.html#resolvers) by default.

**hydra**: Interpolates into the `hydra` config node. e.g. Use `${hydra:job.name}` to get the Hydra job name.

**now**: Creates a string representing the current time using 
[strftime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior).
e.g. for formatting the time you can use something like`${now:%H-%M-%S}`.

**python_version**: Return a string representing the runtime python version by calling `sys.version_info`.
Takes an optional argument of a string with the values major, minor or macro.
e.g:
```yaml
default: ${python_version:}          # 3.8
major:   ${python_version:major}     # 3
minor:   ${python_version:minor}     # 3.8
micro:   ${python_version:micro}     # 3.8.2
```

You can learn more about OmegaConf <a class="external" href="https://omegaconf.readthedocs.io/en/latest/usage.html#access-and-manipulation" target="_blank">here</a>.
