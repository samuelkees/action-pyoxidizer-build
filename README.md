# pyoxidizer-build
GitHub Action for python projects using poetry

## Description
This action will use a python poetry project and build executables/installer for either linux or windows.
Building for mac is currently not yet supported.

## Action Inputs
| Input name | Description | Required | Default Value |
| --- | --- | --- | --- |
| target-system | Select one of the following system to build on: [linux_gnu_x64, windows_x32, windows_x64] | true | None |
| app-name | The name of the app. This name can later be used in the command line to call the app | true | None |
| run-command | The command to run at the start of the app. (eg. \"from package/main import main; main()\" | true | None |
| display-name | The name, which appears in the name of the msi-installer and in the program files folder | true | None |
| app-author | The name of the author or the manufacturer, which is displayed when installing the app per msi-installer | true | None |
| artifact-name | Name of the artifact, where the built file should be uploaded | true | None |
| use-own-pyoxidizer-config | Use own pyoxidizer.bzl, given in the project folder if true, otherwise use standard config provided by this action | false | false |
| python-version | Version of python to install | false | "3.10" |
| poetry-version | Version of poetry to install | false | "1.1.14" |

## Action Outputs
| Output name | Description |
| --- | --- |
| - | - |
## Example
This example will build either an executable for linux (gnu) or and executable & msi-installer for windows (x32 or x64) and download it via the artifact-name.

#### Notes
- No checkout is needed
- For linux: If the linux executable is build on ubuntu-22.04, the executable can't be used on lower ubuntu versions. Therefore, to be most compatible, you have to use ubuntu-20.04. Read more in the [pyoxidizer docs](https://pyoxidizer.readthedocs.io/en/stable/pyoxidizer_distributing_linux.html#managing-binary-portability-on-linux)

```yml
name: Build Linux or Windows

on: 
  push:
    branches:
    - master

env:
  APP_NAME: example_app
  RUN_COMMAND: "from package.main import main; main()"
  DISPLAY_NAME: "Example App"
  APP_AUTHOR: "Your Name / Your Organization"
  ARTIFACT_NAME: example_artifact

jobs:

  build:
    runs-on: ubuntu-20.04   # runs-on: windows-2022
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    
    - name: build executable
      uses: henningwoehr/actions/pyoxidizer-build@main
      with:
        target-system: linux_gnu_x64        # windows_x32 or windows_x64
        app-name: ${{ env.APP_NAME }}
        run-command: ${{ env.RUN_COMMAND }}
        artifact-name: ${{ env.ARTIFACT_NAME }}
        display-name: ${{ env.DISPLAY_NAME }}
        app-author: ${{ env.APP_AUTHOR }}
        python-version: "3.10"
        poetry-version: "1.1.14"

  download_files:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Download files
        uses: actions/download-artifact@v2
        with:
          name: ${{ env.ARTIFACT_NAME }}
          path: built_files

      - name: List files
        run: ls built_files

```