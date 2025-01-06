# GitHub Actions : Reusable and composite actions

- [GitHub Actions : Reusable and composite actions](#github-actions--reusable-and-composite-actions)
  - [Composite action](#composite-action)
    - [Available inputs for the first composite action](#available-inputs-for-the-first-composite-action)
    - [Example for the composite action](#example-for-the-composite-action)
  - [Reusable actions](#reusable-actions)
    - [Example](#example)
  - [Tags guide](#tags-guide)
    - [First number (vX or vX.0.0) : Major update](#first-number-vx-or-vx00--major-update)
    - [Second number (v0.X.0) : Minor update](#second-number-v0x0--minor-update)
    - [Third number (v0.0.X) : Fix update](#third-number-v00x--fix-update)
    - [When updating tags](#when-updating-tags)


## Composite action
You have a composite action to build and install the Arcane Framework.

This composite action allowing you to build and install all of the Framework :
  - `arcaneframework/gh_actions/build_install_framework@v2`

This composite action can work with any docker image with the necessary/recommended Arcane dependencies (see [documentation](https://arcaneframework.github.io/arcane/userdoc/html/d0/d6e/arcanedoc_build_install_prerequisites.html) for more information).

### Available inputs for the first composite action
| Input | Description | Required (Default value) |
| :---: |    :---:    | :---: |
| `source_dir`  | Directory with framework sources. Get the sources with the `actions/checkout` action.  | Yes |
| `build_dir`  | Directory for build files. No need to create this folder before.  | Yes |
| `install_dir`  | Directory for installation files. No need to create this folder before. If it is empty, the installation will not proceed. | No () |
| `log_dir`  | Log directory to save logs. You can use the `actions/upload-artifact` action to easily get the logs. If no value is given, the log will not be moved from the build folder. | No () |
| `cache_dir`  | Cache directory to speed up compilation. You can use the `actions/cache` action to save/restore this folder for a future build. If it is empty, ccache will not be used. | No () |
| `max_size_cache_dir`  | Max size for the `cache_dir` directory. | No (`5G`) |
| `ccache_debug_mode`  | Activate CCache debug mode (need a `log_dir`). | No (`false`) |
| `cmake_additionnal_args`  | Additional arguments given to CMake configure. Example: `'-DARCCORE_CXX_STANDARD=23 -DARCANE_DISABLE_PERFCOUNTER_TESTS=ON -DARCANE_DEFAULT_PARTITIONER=Metis'`  | No () |
| `type_build`  | Type of build. You can choose `Debug`, `Check` or `Release`.  | No (`Release`) |
| `use_ninja`  | Use ninja instead of make to build the Framework.  | No (`true`) |
| `use_shared_libs`  | Generate shared libs instead of static libs.  | No (`true`) |
| `verbose`  | Add verbose args for make/ninja.  | No (`false`) |
| `compilo`  | Compiler to build the Framework. You can choose `gcc` or `clang`. If you want an other compiler, you can use `cmake_additionnal_args` input.  | No (`gcc`) |
| `acc_compilo`  | Compiler to compile GPU part of the framework. You can choose `cuda` (for `nvcc` compiler) or `clang` (to compile CUDA part with `clang++`) or `acpp` (to use AdaptiveCPP).  | No () |
| `with_samples`  | Build samples. Need an `install_dir`. | No (`false`) |
| `with_userdoc`  | Build the user documentation. Available in `build_dir/share/userdoc`. | No (`false`) |
| `with_devdoc`  | Build the dev documentation. Available in `build_dir/share/devdoc`. | No (`false`) |
| `with_aliendoc`  | Build the Alien documentation. Available in `build_dir/share/aliendoc`. | No (`false`) |

### Example for the composite action
https://github.com/arcaneframework/gh_actions/blob/master/.github/workflows/reusable_test_framework.yml
```yml
env:
  # OpenMPI
  OMPI_ALLOW_RUN_AS_ROOT: 1
  OMPI_ALLOW_RUN_AS_ROOT_CONFIRM: 1

jobs:
  build-install-test:
    name: 'Build, install and test Arcane Framework'
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/arcaneframework/ubuntu-2404:gcc-14_minimal_20240717
    steps:
      - name: Define environment paths
        shell: bash
        run: |
          echo "SOURCE_DIR=${GITHUB_WORKSPACE}/framework" >> $GITHUB_ENV
          echo "BUILD_DIR=${GITHUB_WORKSPACE}/build" >> $GITHUB_ENV
          echo "INSTALL_DIR=${GITHUB_WORKSPACE}/install" >> $GITHUB_ENV
          echo "CCACHE_DIR=${GITHUB_WORKSPACE}/ccache" >> $GITHUB_ENV

      - name: Set C++ compiler and default MPI
        shell: bash
        run: |
          source /root/scripts/use_openmpi.sh
          source /root/scripts/use_gcc-14.sh

      - name: Checkout framework
        uses: actions/checkout@v4
        with:
          repository: 'arcaneframework/framework'
          path: ${{ env.SOURCE_DIR }}
          submodules: true

      - name: Build and install framework
        uses: arcaneframework/gh_actions/build_install_framework@v2
        with:
          source_dir: ${{ env.SOURCE_DIR }}
          build_dir: ${{ env.BUILD_DIR }}
          install_dir: ${{ env.INSTALL_DIR }}
          cache_dir: ${{ env.CCACHE_DIR }}
          cmake_additionnal_args: '-DARCCORE_CXX_STANDARD=23 -DARCANE_DISABLE_PERFCOUNTER_TESTS=ON -DARCANE_DEFAULT_PARTITIONER=Metis'
          type_build: Debug
          compilo: gcc
          with_cuda: false
          with_samples: false
```

## Reusable actions
You have a reusable action to test the Arcane Framework :
- `arcaneframework/gh_actions/.github/workflows/reusable_test_framework.yml@v2`

Before using it, you can read this Github Docs page : https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow

There are several 'input' options. To find out the available options, you can read the YAML action file.

This reusable action can work only with [framework-ci](https://github.com/arcaneframework/framework-ci) images. The reusable actions version 1 can work with all framework-ci images v1 and v2 (`20240703` or before). The reusable action version 2 need framework-ci images v3 (`20240717` and after).


### Example
https://github.com/arcaneframework/framework/blob/main/.github/workflows/build_tests_release.yml
```yml
jobs:
  build-install-test:
    name: '[U24_C18_M]_OMPI_Release'
    uses: 'arcaneframework/gh_actions/.github/workflows/reusable_test_framework.yml@v2'
    with:
      image: ghcr.io/arcaneframework/ubuntu-2404:clang-18_minimal_20240717
      compilo_name: clang
      compilo_version: 18
      mpi: OMPI
      cuda: 'false'
      type_build: Release
      cmake_additionnal_args: '-DARCCORE_CXX_STANDARD=23 -DARCANE_DISABLE_PERFCOUNTER_TESTS=ON -DARCANE_DEFAULT_PARTITIONER=Metis'
      with_samples: true
      execute_tests: true
      excluded_tests: ^.*([3-9]proc|[1-9][0-9]+proc|[5-9]thread|[1-9][0-9]+thread).*$
      excluded_tests_with_labels: LARGE_HYBRID
      cache_key_prefix: U24_C18_M_OMPI_Release
```
## Tags guide

There are two type of tags : small tags (e.g. v2) and long tags (e.g. v2.1.0).

### First number (vX or vX.0.0) : Major update

A major update may require new images and may remove, modify or add some options.
These are major changes that require user action.

### Second number (v0.X.0) : Minor update

A minor update can only add a few optional options and/or fix bugs.
There are no major changes.

### Third number (v0.0.X) : Fix update

A fix update can only fix bugs.

### When updating tags
In `.github/workflows/`, action tags need to be updated.

