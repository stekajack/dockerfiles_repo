# {{IMAGE_NAME}}

Container image with my development builds of [ESPResSo](https://espressomd.org) and related tooling
([pressomancy](https://github.com/stekajack/pressomancy) and friends).
Dockerfiles and general usage notes (MPI as root, Apptainer/Singularity on HPC clusters, ...) live in
[stekajack/dockerfiles_repo](https://github.com/stekajack/dockerfiles_repo).

This README is generated on every push; do not edit it on Docker Hub.

## Tags

<!-- TAGS -->

<!-- TAG-BLOCK-TEMPLATE
### `{{IMAGE_TAG}}`

Pushed {{PUSH_DATE}} · built {{BUILD_DATE}} · {{OS}} · Python {{PYTHON_VERSION}} · {{IMAGE_SIZE}}

Dockerfile: {{DOCKERFILE_LINK}}

| package | source | branch | commit | commit date | local changes |
|---|---|---|---|---|---|
| espresso | {{ESPRESSO_SOURCE}} | {{ESPRESSO_BRANCH}} | {{ESPRESSO_COMMIT_LINK}} | {{ESPRESSO_DATE}} | {{ESPRESSO_PATCHED}} |
| pressomancy | {{PRESSOMANCY_SOURCE}} | {{PRESSOMANCY_BRANCH}} | {{PRESSOMANCY_COMMIT_LINK}} | {{PRESSOMANCY_DATE}} | {{PRESSOMANCY_PATCHED}} |
| pressomancy-toolbox | {{PRESSOMANCY_TOOLBOX_SOURCE}} | {{PRESSOMANCY_TOOLBOX_BRANCH}} | {{PRESSOMANCY_TOOLBOX_COMMIT_LINK}} | {{PRESSOMANCY_TOOLBOX_DATE}} | {{PRESSOMANCY_TOOLBOX_PATCHED}} |
| espressoSq | {{ESPRESSOSQ_SOURCE}} | {{ESPRESSOSQ_BRANCH}} | {{ESPRESSOSQ_COMMIT_LINK}} | {{ESPRESSOSQ_DATE}} | {{ESPRESSOSQ_PATCHED}} |

ESPResSo CMake ({{ESPRESSO_BUILD_TYPE}}) ON: {{ESPRESSO_CMAKE_ON}}

ESPResSo CMake OFF: {{ESPRESSO_CMAKE_OFF}}

`espressomd.features()`: {{ESPRESSO_FEATURES}}
TAG-BLOCK-TEMPLATE -->
