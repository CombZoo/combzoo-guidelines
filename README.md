# combzoo-guidelines

*Guidelines for the Combinatorial Optimization Zoo - CombZoo*

This repository introduces CombZoo, its foundations and rules for integrating new solvers.

## purpose

[CombZoo](github.com/combzoo) intends to organize a pack of solvers for challenging combinatorial optimization problems, many of them belonging to the NP-Hard class. The name is partially inspired by another zoo, the [Complexity Zoo](https://complexityzoo.net), which uses MediaWiki technology to organize theoretical results on complexity (visit letter [N](https://complexityzoo.net/Complexity_Zoo:N) to read more on NP and NP-Complete classes).
On the other hard, CombZoo is focused on providing *algorithmic implementations of solvers* for combinatorial optimization problems (although other kinds of optimization problems are also accepted), thus providing basic coding guidelines to integrate and interoperate with new solvers (specially focused on C/C++ languages).

The main goal of CombZoo is to promote Open Science, so as giving learning opportunities and access to state-of-the-art optimization solvers for the general public (without needing to request permission for authors).

## coding guidelines

To be able to interoperate with other solvers, we require that solvers implement two functions (`solve` and `info`) as a C-style library interface:

```{.cpp}
extern "C"
{
   int solver_kp01_generic(int format, const char* config, char* output);

   int info_kp01_generic(int format, const char* key, char* output);

} // extern "C"
```

The naming convention for the functions is the following:
- prefix (`solver` or `info`)
- problem name (in the above example, `kp01`)
- solver name (in the above example, `generic`)

The return `int` value corresponds to standard error code (`=0` if all fine).

### function `info`

Function `info` provides basic information regarding solver capabilities, writing these on output according to specific `format`. Currently, we adopt `format=1` for "standard json", and leave `=0` unused, intending to represent "default". Other possibility is to accept some sort of "plaintext" as `format=2`, if necessary. `key` parameter can be used to filter the output (thus not returning the complete "info", if too long output).

An example of "info" intended for project [solver_kp01_generic](github.com/combzoo/solver_kp01_generic):
```{.json}
{
      "standard": 0,
      "name": "solver_kp01_generic",
      "description": "generic solver for the 01 knapsack problem",
      "author" : "CombZoo Team",
      "version": "0.1.0",
      "license": "MIT",
      "language": "cpp",
      "type": "meta",
      "configurations": []
}
```

This specification is currently not stable (`standard=0`), so feel free to suggest improvements, as long as more problems are supported. The `configurations` section is intended to describe the allowed launch configurations for the solver, including parameter names, types, limiting ranges (max/min) and recommended ranges (max/min).
The type "meta" stands for "meta-solver", meaning that it may invoke delegated solvers for specific problem cases (useful for very classic problems). It could typically be "exact" or "heuristic", although we leave this field quite flexible for now.

### function `solve`

Function `solve` invokes the solver for the given target problem (see `problem_file`), by means of a configuration file. This configuration file also includes parameters for the algorithm, timelimit conditions, and stop criteria on general.
The problem can also be passed directly, in some `problem` field, according to some `problem_type` description (useful for diverse problem representations on each scenario).
We intend to support more experimentation-driven features (still `standard=0`), such as greater `batch` size, e.g., to repeat experimentation and compute average, stddev, etc, in order to incentive better experimentation practices for overall community.

An example for configuration file (according to [solver_kp01_generic](github.com/combzoo/solver_kp01_generic)):

```{.json}
{
    "standard" : 0,
    "description": "default kp01 launch configuration - based on metaheuristic Simulated Annealing",
    "problem_file": "knapsack.txt",
    "seed": 1234999991,
    "timelimit": 10.0,
    "batch": 1,
    "log_level" : "default",
    "strategy": {
        "name": "BasicSimulatedAnnealing",
        "alpha": 0.98,
        "Tmax": 99999,
        "iterT": 100
    }
}
```

## Command-line parameters

Another possibility is to create a binary for solver with a common command-line interface, thus not exposing source-code (in proprietary projects).

In this case, recommend the following options:

- `--config config_file.json`
- `--info [0]`
- `--help`

Option `--info` returns information for the solver, in the specified format ("0" is standard, which is "1" for json standard). Option `--config` executes the solver with the passed configuration file. The target problem and algorithmic parameters are also specified in the config_file (extension may be used to differentiate between evolving standards).
Option `--help` displays the help, as usual.

An example for [solver_kp01_generic](github.com/combzoo/solver_kp01_generic):

```{.bash}
# build with bazel
bazel build ...   
# execute solver     
./bazel-bin/app_KP01_generic --config default_kp01.json
```

## extensions

We plan to extend this document with guidelines for other popular languages (such as Python, Java, etc), since many solvers are currently designed in these languages. The general advice for command-line already applies for these languages, but some library-oriented is also welcome, since this allows solving process to be performed without any disk access (no read/write files).

We currently lack some specific standard for the output of solvers, although [solver_kp01_generic](github.com/combzoo/solver_kp01_generic) may already give some ideas on that:

```{.json}
{
  "best_value" : 7,
  "status" : 0,
  "time_spent" : 0.24
}
```

## license

CombZoo is a free project, so are all solvers submitted to this project. Free licenses can be accepted but limiting side effects may happen with some GNU licenses, so MIT License / Creative Commons are recommended on general.
