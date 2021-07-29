# Thonny Flatpak

This is the official Thonny Flatpak.

## Users

This section covers installation instructions and additional information for Thonny users.

### Install

Add the Flathub remote.

    flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Install the Thonny Flatpak.

    flatpak install flathub org.thonny.Thonny

## Maintainers

### Build

Get the source code.

    git clone https://github.com/flathub/org.thonny.Thonny.git
    cd org.thonny.Thonny/packaging/linux

Add the Flathub repository.

    flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Install the FreeDesktop SDK and Platform.

    flatpak install --user flathub org.freedesktop.Sdk//20.08 org.freedesktop.Platform//20.08

Install Flatpak Builder.

    sudo apt install flatpak-builder

Build the Flatpak.

    flatpak-builder --user --install --force-clean --repo=repo thonny-flatpak-build-dir org.thonny.Thonny.yaml

Run the Flatpak.

    flatpak run org.thonny.Thonny

### Update

The Python dependencies for the Flatpak are generated with the [Flatpak Pip Generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/pip).
This tool is used to produces the `python3-modules.json` and `python3-wheel.json` files, which are included in the Flatpak manifest.
In order to update the dependencies used in the Flatpak, these files must be regenerated.
Follow the instructions here to do so.

First, install the Python dependency `requirements-parser`.

    python3 -m pip install requirements-parser

Clone the [Flatpak Builder Tools](https://github.com/flatpak/flatpak-builder-tools) repository.
Currently, it's necessary to use [a fork](https://github.com/nanonyme/flatpak-builder-tools) of the project which allows building all the Python dependencies without throwing errors.

    git clone https://github.com/nanonyme/flatpak-builder-tools.git
    git -C flatpak-builder-tools switch support-build-isolation

Run the Flatpak Pip Generator script in the `packaging/linux` directory to produce an updated `python3-modules.json` manifest and an updated `python3-wheel.json` manifest.
The dependencies for the `python3-modules.json` manifest are retrieved from the `requirements-regular-bundle.txt` and `requirements-xxl-bundle.txt` files.

    python3 flatpak-builder-tools/pip/flatpak-pip-generator \
        --runtime org.freedesktop.Sdk//20.08 \
        wheel
    wget -L https://raw.githubusercontent.com/thonny/thonny/master/packaging/requirements-regular-bundle.txt
    wget -L https://raw.githubusercontent.com/thonny/thonny/master/packaging/requirements-xxl-bundle.txt
    python3 flatpak-builder-tools/pip/flatpak-pip-generator \
        --runtime org.freedesktop.Sdk//20.08 \
        $(cat requirements-regular-bundle.txt) \
        $(cat requirements-xxl-bundle.txt)

If you have `org.freedesktop.Sdk//20.08` installed in *both* the user and system installations, the Flatpak Pip Generator will choke generating the manifest.
The best option at the moment is to temporarily remove either the user or the system installation until this issue is fixed upstream.
