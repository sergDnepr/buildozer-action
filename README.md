# Buildozer action

[![Build workflow](https://github.com/ArtemSBulgakov/buildozer-action/workflows/Build/badge.svg?branch=master)](https://github.com/ArtemSBulgakov/buildozer-action/actions?query=workflow%3ABuild)
[![Build (with Buildozer master) workflow](<https://github.com/ArtemSBulgakov/buildozer-action/workflows/Build%20(with%20Buildozer%20master)/badge.svg?branch=master>)](https://github.com/ArtemSBulgakov/buildozer-action/actions?query=workflow%3A%22Build+%28with+Buildozer+master%29%22)

Build your Python/[Kivy](https://github.com/kivy/kivy) applications for Android
with [Buildozer](https://github.com/kivy/buildozer). This action uses official
Buildozer [Docker image](https://github.com/kivy/buildozer/blob/master/Dockerfile),
but adds some features and patches to use in GitHub Actions.

## Inputs

### `command`

**Required** Command to start Buildozer.

- _Default:_ `buildozer android debug` _(iOS and OSX is not supported because Docker cannot run on MacOS)_.
- For more commands use `;` as delimiter: `python3 setup.py build_ext --inplace; buildozer android debug`.

### `workdir`

**Required** Working directory where buildozer.spec is located.

- _Default:_ `.` (top directory).
- Set to `src` if buildozer.spec is in `src` directory.

### `buildozer_version`

**Required** Version of Buildozer to install.

- _Default:_ `stable` (latest release on PyPI, `pip install buildozer`).
- Set to `master` to use [master](https://github.com/kivy/buildozer/tree/master) branch _(`pip install git+https://github.com/kivy/buildozer.git@master`)_.
- Set to [tag](https://github.com/kivy/buildozer/tree/1.2.0) name `1.2.0` to use specific release _(`pip install git+https://github.com/kivy/buildozer.git@1.2.0`)_.
- Set to [commit](https://github.com/kivy/buildozer/tree/94cfcb8) hash `94cfcb8` to use specific commit _(`pip install git+https://github.com/kivy/buildozer.git@94cfcb8`)_.
- Set to git+ address `git+https://github.com/username/buildozer.git@master` to use fork.
- Set to directory name `./my_buildozer` to install from local path _(`pip install ./my_buildozer`)_.
- Set to nothing `''` to not install buildozer

## Outputs

### `filename`

Filename of built package relative to repository root.

- Example: `test_app/bin/testapp-0.1-armeabi-v7a-debug.apk`

## Environment variables

You can set environment variables to change Buildozer settings. See
[Buildozer Readme](https://github.com/kivy/buildozer#default-config) for more
information.

Example (change Android architecture):

```yaml
env:
  APP_ANDROID_ARCH: armeabi-v7a
```

## Caching

You can set up cache for Buildozer global and local directories. Global
directory is in root of repository. Local directory is in workdir.

- Global: `.buildozer-global` (sdk, ndk, platform-tools)
- Local: `test_app/.buildozer` (dependencies, build temp, _not recommended to cache_)

I don't recommend to cache local buildozer directory because Buildozer doesn't
automatically update dependencies to latest version.

Use cache only if it speeds up your workflow! Usually this only adds 1-3 minutes
to job running time, so I don't use it.

Example:

```yaml
- name: Cache Buildozer global directory
  uses: actions/cache@v2
  with:
    path: .buildozer_global
    key: buildozer-global-${{ hashFiles('test_app/buildozer.spec') }} # Replace with your path
```

## Example usage

```yaml
- name: Build with Buildozer
  uses: ArtemSBulgakov/buildozer-action@v1
  id: buildozer
  with:
    command: buildozer android debug
    workir: src
    buildozer_version: stable
```

## Full workflow

```yaml
name: Build
on: [push, pull_request]

jobs:
  # Build job. Builds app for Android with Buildozer
  build-android:
    name: Build for Android
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Build with Buildozer
        uses: ArtemSBulgakov/buildozer-action@v1
        id: buildozer
        with:
          workdir: test_app
          buildozer_version: stable

      - name: Upload artifacts
        uses: actions/upload-artifact@v2
        with:
          name: package
          path: ${{ steps.buildozer.outputs.filename }}
```

## Action versioning

Currently it is recommended to use `v1` tag. This tag updates when new `v1.x.x`
version released. All `v1` versions will have backward compatibility. You will
get warning when `v2` will be released.

## How to build packages locally

Use official Buildozer's [Docker image](https://hub.docker.com/r/kivy/buildozer)
([repository](https://github.com/kivy/buildozer#buildozer-docker-image)).

## Contributing

Create Bug Request if you have problems with running this action or
Feature Request if you have ideas how to improve it. If you know how to fix
something, feel free to fork repository and create Pull Request. Test your
changes in fork before creating Pull Request.

## License

ArtemSBulgakov/buildozer-action is released under the terms of the
[MIT License](LICENSE).
