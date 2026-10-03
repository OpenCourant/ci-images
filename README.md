# OpenCourant CI images

Container images used by OpenCourant CI, built from public sources and hosted
in the OpenCourant GitHub namespace at `ghcr.io/opencourant`.

These images replace the Altair-internal images referenced by the CI
workflows inherited from OpenRadioss
(`internal.artifacts.altair.com/radioss-docker-local/...`), which are not
accessible outside Altair infrastructure.

## Images

### `ghcr.io/opencourant/linux64-bld-qa`

Linux x86_64 build + QA environment for Starter and Engine:

| Component  | Version                     | Source                                   |
|------------|-----------------------------|------------------------------------------|
| Base OS    | Rocky Linux 8.10            | `docker.io/rockylinux/rockylinux:8.10`   |
| Toolchain  | GCC/GFortran/G++ 11         | `gcc-toolset-11` (AppStream)             |
| MPI        | OpenMPI 4.1.2 (`/opt/openmpi`) | Built from source (SHA-256 pinned)    |
| Build      | CMake, GNU make             | AppStream                                |
| QA         | Perl, Python 3              | AppStream                                |

Tags:

- `ompi4.1.2` — moving tag, latest rebuild of this environment
- `ompi4.1.2-YYYYMMDD` — immutable date-stamped builds
- `latest`

The environment matches the toolchain documented in the OpenCourant
`HOWTO.md` (gcc-toolset-11 on EL8, OpenMPI 4.1.2 in `/opt/openmpi`). The CI
workflows `source /root/.bashrc`, which enables the toolset and puts OpenMPI
on `PATH`/`LD_LIBRARY_PATH`.

## Building locally

```sh
podman build -t ghcr.io/opencourant/linux64-bld-qa:ompi4.1.2 linux64-bld-qa
# or
docker build -t ghcr.io/opencourant/linux64-bld-qa:ompi4.1.2 linux64-bld-qa
```

## Publishing

Images are built and pushed to GHCR automatically by
[`.github/workflows/build-push.yml`](.github/workflows/build-push.yml) on
pushes to `main` that touch an image directory, or manually via
`workflow_dispatch`.
