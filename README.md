# GitHub Actions : Reusable and composite actions

## Reusable actions
You have two reusable actions to test the Arcane Framework :
- `arcaneframework/gh_actions/.github/workflows/reusable_test_framework.yml@v1`
- `arcaneframework/gh_actions/.github/workflows/reusable_test_split_framework.yml@v1`

Before using, you can read this Github Docs page : https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow

There are some 'input' options. To get available options, you can read the YAML action file.


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


## Composite actions
You have six composite actions to build and install the Arcane Framework.

- One composite action allows to build and install all of the Framework. It's the recommended method to use the Framework :
  - `arcaneframework/gh_actions/build_install_framework@v1`
- Five composite actions are available to build all of the part of the Framework (one action per part) (soon deprecated) :
  - `arcaneframework/gh_actions/split/install_arccon@v1`
  - `arcaneframework/gh_actions/split/install_arcdependencies@v1`
  - `arcaneframework/gh_actions/split/build_install_axlstar@v1`
  - `arcaneframework/gh_actions/split/build_install_arccore@v1`
  - `arcaneframework/gh_actions/split/build_install_arcane@v1`

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
