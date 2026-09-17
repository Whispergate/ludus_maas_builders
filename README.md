# ludus_maas_builders

Ansible role that builds Docker toolchain images for the MAAS payload compilation pipeline. These images are used by Jenkins Docker Pipeline to run containerised builds.

## What it does

1. Creates a build directory at `/opt/maas-builders`
2. Creates a shared `/payloads` volume for artifact collection across runners
3. Templates Dockerfiles for each builder image
4. Builds the Docker images locally on the runner host

## Builder Images

| Image | Contents |
|-------|----------|
| `maas-builder-c` | Debian 12, MinGW (x86_64), Clang, LLD, LLVM, NASM, CMake, Python 3 + pefile |
| `maas-builder-go` | Go 1.26, Garble (latest), MinGW (for CGO cross-compilation) |
| `maas-builder-nim` | Nim (latest via choosenim), MinGW (x86_64), winim library |
| `maas-builder-rust` | Rust 1.82, x86_64-pc-windows-gnu target, MinGW (x86_64) |

## Requirements

- Debian 12 or Ubuntu 22.04/24.04
- Docker Engine installed (the `ludus_jenkins_agent` role handles this)
- Internet access during image build (for pulling base images and packages)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ludus_maas_builder_images` | `[maas-builder-c, maas-builder-go, maas-builder-rust, maas-builder-nim]` | List of images to build |
| `ludus_maas_builder_dir` | `/opt/maas-builders` | Directory for Dockerfiles |
| `ludus_maas_shared_volume` | `/payloads` | Shared volume path for build artifacts |

## Required Ansible Collections

- `community.docker`

## Example (Ludus range config)

```yaml
- vm_name: '{{ range_id }}-runner-lin01'
  hostname: '{{ range_id }}-runner-lin01'
  template: debian-12-x64-server-template
  vlan: 99
  ip_last_octet: 4
  ram_gb: 8
  cpus: 4
  linux: true
  roles:
    - whispergate.ludus_jenkins_agent
    - whispergate.ludus_maas_builders
  role_vars:
    ludus_maas_builder_images:
      - maas-builder-c
      - maas-builder-go
      - maas-builder-rust
```

## Customisation

To add a new toolchain, create a `Dockerfile.maas-builder-<name>.j2` template in the role's `templates/` directory and add the image name to `ludus_maas_builder_images`.

## License

BSD-2-Clause
