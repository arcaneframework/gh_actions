# GitHub Actions : Reusable and composite actions

## Composite actions
You have six composite actions to build and install the Arcane Framework.

- One composite action allowing you to build and install all of the Framework. This is the recommended method to use the Framework :
  - `arcaneframework/gh_actions/build_install_framework@v1`
- Five composite actions are available to build all Framework parts (one action per part) (soon deprecated) :
  - `arcaneframework/gh_actions/split/install_arccon@v1`
  - `arcaneframework/gh_actions/split/install_arcdependencies@v1`
  - `arcaneframework/gh_actions/split/build_install_axlstar@v1`
  - `arcaneframework/gh_actions/split/build_install_arccore@v1`
  - `arcaneframework/gh_actions/split/build_install_arcane@v1`

### Available inputs for the first composite action:
| Input | Description | Required (Default value) |
| :---: |    :---:    | :---: |
| `source_dir`  | Directory with framework sources. Get the sources with the `actions/checkout` action.  | Yes |
| `build_dir`  | Directory for build files. No need to create this folder before.  | Yes |
| `install_dir`  | Directory for installation files. No need to create this folder before. If it is empty, the installation will not proceed. | No () |
| `cache_dir`  | Cache directory to speed up compilation. You can use the `actions/cache` action to save/restore this folder for a future build. If it is empty, ccache will not be used. | No () |
| `log_dir`  | Log directory to save logs. You can use the `actions/upload-artifact` action to easily get the logs. If no value is given, the log will not be moved from the build folder. | No () |
| `cmake_additionnal_args`  | Additional arguments given to CMake configure. Example: `'-DARCCORE_CXX_STANDARD=23 -DARCANE_DISABLE_PERFCOUNTER_TESTS=ON -DARCANE_DEFAULT_PARTITIONER=Metis'`  | No () |
| `type_build`  | Type of build. You can choose `Debug`, `Check` or `Release`.  | No (`Release`) |
| `use_ninja`  | Use ninja instead of make to build the Framework.  | No (`true`) |
| `use_shared_libs`  | Generate shared libs instead of static libs.  | No (`true`) |
| `verbose`  | Add verbose args for make/ninja.  | No (`false`) |
| `compilo`  | Compiler to build the Framework. You can choose `GCC` or `Clang`. If you want an other compiler, you can use `cmake_additionnal_args` input.  | No (`GCC`) |
| `with_cuda`  | Use CUDA to compile GPU part of the framework. Need `nvcc` compiler.  | No (`false`) |
| `with_samples`  | Build samples. Need an `install_dir`. | No (`false`) |

### Example for the first composite action:
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
      image: ghcr.io/arcaneframework/ubuntu-2204:gcc-12_clang-16_minimal_20230808
    steps:
      - name: Define environment paths
        shell: bash
        run: |
          echo "SOURCE_DIR=${GITHUB_WORKSPACE}/framework" >> $GITHUB_ENV
          echo "BUILD_DIR=${GITHUB_WORKSPACE}/build" >> $GITHUB_ENV
          echo "INSTALL_DIR=${GITHUB_WORKSPACE}/install" >> $GITHUB_ENV
          echo "CCACHE_DIR=${GITHUB_WORKSPACE}/ccache" >> $GITHUB_ENV

      - name: Checkout framework
        uses: actions/checkout@v3
        with:
          repository: 'arcaneframework/framework'
          path: ${{ env.SOURCE_DIR }}
          submodules: true

      - name: Build and install framework
        uses: arcaneframework/gh_actions/build_install_framework@v1
        with:
          source_dir: ${{ env.SOURCE_DIR }}
          build_dir: ${{ env.BUILD_DIR }}
          install_dir: ${{ env.INSTALL_DIR }}
          cache_dir: ${{ env.CCACHE_DIR }}
          cmake_additionnal_args: '-DARCCORE_CXX_STANDARD=23 -DARCANE_DISABLE_PERFCOUNTER_TESTS=ON -DARCANE_DEFAULT_PARTITIONER=Metis'
          type_build: Debug
          compilo: GCC
          with_cuda: false
          with_samples: false
```

### Example for the last composite actions:
https://github.com/arcaneframework/gh_actions/blob/master/.github/workflows/reusable_test_split_framework.yml

## Reusable actions
You have two reusable actions to test the Arcane Framework :
- `arcaneframework/gh_actions/.github/workflows/reusable_test_framework.yml@v1`
- `arcaneframework/gh_actions/.github/workflows/reusable_test_split_framework.yml@v1`

Before using it, you can read this Github Docs page : https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow

There are several 'input' options. To find out the available options, you can read the YAML action file.


### Example:
https://github.com/arcaneframework/framework/blob/main/.github/workflows/build_tests_release.yml
```yml
jobs:
  build-install-test:
    name: '[U22_G12_C16_M]_CLang_OpenMPI_Release'
    uses: 'arcaneframework/gh_actions/.github/workflows/reusable_test_framework.yml@v1'
    with:
      image: ghcr.io/arcaneframework/ubuntu-2204:gcc-12_clang-16_minimal_20230808
      compilo: CLang
      mpi: OpenMPI
      with_cuda: false
      type_build: Release
      cmake_additionnal_args: '-DARCCORE_CXX_STANDARD=23 -DARCANE_DISABLE_PERFCOUNTER_TESTS=ON -DARCANE_DEFAULT_PARTITIONER=Metis'
      with_samples: true
      execute_tests: true
      excluded_tests: ^.*([3-9]proc|[1-9][0-9]+proc|[5-9]thread|[1-9][0-9]+thread).*$
      excluded_tests_with_labels: LARGE_HYBRID
      cache_key_prefix: U22_G12_C16_M_CLang_OpenMPI_Release
```
