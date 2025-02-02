# buildtest-cori

This repository is [Cori](https://docs.nersc.gov/) Testsuite with buildtest. A mirror of this repository is located at https://software.nersc.gov/siddiq90/buildtest-cori used for running gitlab CI specified in file [.gitlab-ci.yml](https://github.com/buildtesters/buildtest-cori/blob/devel/.gitlab-ci.yml).



## Setup

To get started clone this repo on Cori as follows:

```
git clone https://github.com/buildtesters/buildtest-cori
```

buildtest configuration file is read at `$HOME/.buildtest` if you don't have a directory please run

```
mkdir $HOME/.buildtest
```

Next, navigate to `buildtest-cori` directory and copy your site configuration to the following location

```
cd buildtest-cori
cp .config.yml $HOME/.buildtest/config.yml
```

Assuming you have [installed buildtest](https://buildtest.readthedocs.io/en/devel/installing_buildtest.html) you 
can view and validate your configuration via:

```
buildtest config validate
buildtest config view
```

Please make sure you are using tip of `devel` with buildtest when writing tests. You should sync your local `devel` with upstream
fork, for more details see https://buildtest.readthedocs.io/en/devel/contributing/setup.html

First time around you should discover all buildspecs this can be done via ``buildtest buildspec find``.  The command below will find
and validate all buildspecs in the buildtest-cori repo and load them in buildspec cache.

```
$ buildtest buildspec find --root /path/to/buildtest-cori/buildspecs
```

The buildspecs are loaded in buildspec cache file (JSON) that is used by `buildtest buildspec find` for querying cache. Subsequent runs will
read from cache.  For more details see [buildspec interface](https://buildtest.readthedocs.io/en/devel/gettingstarted/buildspecs_interface.html)


## Building Tests

**Note: All tests are written in YAML using .yml extension**
   
To build tests use ``buildtest build`` command for example we build all tests in ``system`` directory as follows

```
buildtest build -b system/
```

You can specify multiple buildspecs either files or directory via ``-b`` option

```
buildtest build -b slurm/partition.yml -b slurmutils/
```

You can exclude a buildspec via `-x` option this behaves same way as `-b` option so you can specify
a directory or filepath which could be absolute path, or relative path. This is useful when
you want to run multiple tests grouped in directory but exclude a few.

```
buildtest build -b slurm -x slurm/sinfo.yml
```

buildtest can run tests via tags which can be useful when grouping tests, to see a list of available tags you 
can run: ``buildtest buildspec find --tags``



For instance if you want to run all ``lustre`` tests you can use ``buildtest build --tags`` option. 

```
   $ buildtest build --tags lustre

   +-------------------------------+
   | Stage: Discovering Buildspecs |
   +-------------------------------+ 


   Discovered Buildspecs:

   /global/u1/s/siddiq90/buildtest-cori/filesystem/lustre.yml

   +---------------------------+
   | Stage: Parsing Buildspecs |
   +---------------------------+ 

    schemafile              | validstate   | buildspec
   -------------------------+--------------+------------------------------------------------------------
    script-v1.0.schema.json | True         | /global/u1/s/siddiq90/buildtest-cori/filesystem/lustre.yml

   +----------------------+
   | Stage: Building Test |
   +----------------------+ 

    name                | id       | type   | executor   | tags                     | testpath
   ---------------------+----------+--------+------------+--------------------------+-----------------------------------------------------------------------------------------------------
    lustre_osts         | b487c36d | script | local.bash | ['filesystem', 'lustre'] | /global/u1/s/siddiq90/buildtest/var/tests/local.bash/lustre/lustre_osts/1/stage/generate.sh
    lustre_mdts         | 03259756 | script | local.bash | ['filesystem', 'lustre'] | /global/u1/s/siddiq90/buildtest/var/tests/local.bash/lustre/lustre_mdts/1/stage/generate.sh
    lustre_nstaff_quota | ad748ead | script | local.bash | ['filesystem', 'lustre'] | /global/u1/s/siddiq90/buildtest/var/tests/local.bash/lustre/lustre_nstaff_quota/1/stage/generate.sh

   +----------------------+
   | Stage: Running Test  |
   +----------------------+ 

    name                | id       | executor   | status   |   returncode | testpath
   ---------------------+----------+------------+----------+--------------+-----------------------------------------------------------------------------------------------------
    lustre_osts         | b487c36d | local.bash | PASS     |            0 | /global/u1/s/siddiq90/buildtest/var/tests/local.bash/lustre/lustre_osts/1/stage/generate.sh
    lustre_mdts         | 03259756 | local.bash | PASS     |            0 | /global/u1/s/siddiq90/buildtest/var/tests/local.bash/lustre/lustre_mdts/1/stage/generate.sh
    lustre_nstaff_quota | ad748ead | local.bash | PASS     |            0 | /global/u1/s/siddiq90/buildtest/var/tests/local.bash/lustre/lustre_nstaff_quota/1/stage/generate.sh

   +----------------------+
   | Stage: Test Summary  |
   +----------------------+ 

   Executed 3 tests
   Passed Tests: 3/3 Percentage: 100.000%
   Failed Tests: 0/3 Percentage: 0.000%

```

## Tags Breakdown

When you write buildspecs, please make sure you attach one or more `tags` to the test that way your test will get picked up during one of the CI checks. Shown
below is a summary of tag description

- **daily** - this tag is used for running daily system checks using gitlab CI. Tests should run relatively quick
- **system** - this tag is used for classifying all system tests that may include: system configuration, servers, network, cray tests. This tag should be used 
- **slurm** - this tag is used for slurm test that includes slurm utility check, slurm controller, etc... This tag **shouldn't** be used for job submission that is managed by **jobs** tag. The `slurm` tag tests should be short running test that use a Local Executor.
- **jobs** - this tag is used for testing slurm policies by submitting jobs to scheduler. 
- **compile** - this tag is used for compilation of application (OpenMP, MPI, OpenACC, CUDA, upc, bupc, etc...)
- **module** - this tag is used for testing module system
- **benchmark** - this tag is used for benchmark tests. This can be application benchmarks, mini-benchmarks, kernels, etc... 

## Querying Tests

You can use ``buildtest report`` and ``buildtest inspect`` to query tests. The commands differ slightly and data is represented differently. The ``buildtest report`` command will show output in tabular form and only show some of the metadata, if you want to access the entire test record use ``buildtest inspect`` command which displays the content in JSON format. For more details on querying tests see https://buildtest.readthedocs.io/en/devel/gettingstarted/query_test_report.html


## CI Setup

There is a github workflow [.mirror_to_cori.yml](https://github.com/buildtesters/buildtest-cori/blob/devel/.github/workflows/mirror_to_cori.yml) responsible for mirroring upstream project to https://software.nersc.gov/siddiq90/buildtest-cori.  We have setup a gitlab [secret](https://github.com/buildtesters/buildtest-cori/settings/secrets/actions) that contains gitlab personal access token created from https://software.nersc.gov/-/profile/personal_access_tokens. The Personal access token must have `read_api`, `read_repository`, `write_repository` scope.  

Tests are run on schedule basis with one schedule corresponding to one gitlab job in `.gitlab-ci.yml`. The scheduled pipelines are configured in 
https://software.nersc.gov/siddiq90/buildtest-cori/-/pipeline_schedules. Each schedule has a variable defined to control which pipeline 
is run. In the `.gitlab-ci.yml` we make use of conditional rules using [only](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic). For example the daily
system test has variable defined `DAILYCHECK` set to `True` and the gitlab job is defined as follows:


```
scheduled_system_check:
  stage: test
  only:
    refs:
      - schedules
    variables:
      - $DAILYCHECK == "True"
```

The scheduled jobs are run at different intervals (1x/day, 1x/week, etc...) at different times of day to avoid overloading the system. The gitlab jobs
will run jobs based on tags, alternately some tests may be defined by running all tests in a directory (`buildtest build -b apps`). If you want to add a new
scheduled job, please define a [new schedule](https://software.nersc.gov/siddiq90/buildtest-cori/-/pipeline_schedules/new) with an appropriate time. The `target branch` should be `devel` and define a unique variable used to distinguish scheduled jobs. Next, create a job in `.gitlab-ci.yml` that references the scheduled job based on the variable name.


The `validate_tests` gitlab job is responsible for validating buildspecs, please review this job when contributing tests. The buildspec must pass validation
in order for buildtest to build and run the test. 

## Integrations

buildtest-cori mirror repo has integration with Github and Slack. The integrations can be found at https://software.nersc.gov/siddiq90/buildtest-cori/-/settings/integrations. The Github integration will push result back to upstream project: https://github.com/buildtesters/buildtest-cori. The CI checks are pushed to [buildtest Slack](https://hpcbuildtest.slack.com) at **#cori-testsuite** workspace. 

## CDASH

buildtest will push test results to [CDASH](https://www.kitware.com/cdash/project/about.html) server at https://my.cdash.org/index.php?project=buildtest-cori using the script [buildtest_cdash.py](https://github.com/buildtesters/buildtest-cori/blob/devel/buildtest_cdash.py). This script is run in gitlab pipeline after job completion where it processes the file **$BUILDTEST_ROOT/var/report.json**. If you want to debug the script you just need to run the script `python buildtest_cdash.py` as is. The file **$BUILDTEST_ROOT/var/report.json** is created upon building a test (`buildtest build`).

## Contributing Guide

To contribute back you will want to make sure your buildspec is validated before you contribute back, this could be 
done by running test manually ``buildtest build`` or see if buildspec is valid via ``buildtest buildspec find``. It 
would be good to run your test and make sure it is working as expected, you can view test detail using ``buildtest inspect <ID>`` 
or see [Test Inspection](https://buildtest.readthedocs.io/en/devel/getting_started.html#test-inspection) section. 

buildtest relies on json schema to validate buildspecs and you will need to understand the json schema to understand how
to write valid tests. To get all schemas run the following:
```
$ buildtest schema
global.schema.json
definitions.schema.json
settings.schema.json
compiler-v1.0.schema.json
script-v1.0.schema.json
```

The schemas ``script``, ``compiler``, ``global`` are of interest when writing buildspec to view the json content for script 
schema you can run ``buildtest schema -n script-v1.0.schema.json --json``.

To view all examples for script schema you can run ``buildtest schema -n script-v1.0.schema.json --examples`` which will show
all invalid/valid tests. buildtest will validate each example before it shows content which will be help understand how to write
tests and see all the edge cases.

Alternately you can see all published schemas and examples on https://buildtest.readthedocs.io/en/devel/buildspecs/schema_examples.html

If you want to contribute your tests, please see [CONTRIBUTING.md](https://github.com/buildtesters/buildtest-cori/blob/devel/CONTRIBUTING.md)


## References

- buildtest documentation: https://buildtest.readthedocs.io/en/devel/
- buildtest schema docs: https://buildtesters.github.io/buildtest/
- Getting Started: https://buildtest.readthedocs.io/en/devel/getting_started.html
- Writing Buildspecs: https://buildtest.readthedocs.io/en/devel/writing_buildspecs.html
- Contributing Guide: https://buildtest.readthedocs.io/en/devel/contributing.html


