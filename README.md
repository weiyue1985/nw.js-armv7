NW.js port for ARM
============================= 

[NW.js](http://nwjs.io/ "NW.js web site") (formely node-webkit) binary compiled for the **ARMv7**.


## Versions
NW.js version: v0.26.5 

Chromium 62 + Node 9.1.0


## Tested on
* NXP i.MX6 Quad Processor with 2GB RAM using *IMXLinux BSP L4.1.15_2.1.0*

## Building NW.js for Linux ARMv7

### Prerequisites

Read [Building NW.js] tutorial before you go further.

Checkout and install the [depot_tools package]. This contains the custom tools necessary to checkout and build NW.js.

Clone the depot_tools repository:
```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

Add depot_tools to the end of your PATH (you will probably want to put this in your ~/.bashrc or ~/.zshrc). Assuming you cloned depot_tools to /path/to/depot_tools:
```bash
export PATH=$PATH:/path/to/depot_tools
```

### Get the Code

**Step 1.** Create a folder for holding NW.js source code, export it to `NWJS` environment variable, and run following command in the folder to generate `.gclient` file:

```bash
mkdir -p $HOME/nwjs
export NWJS=$HOME/nwjs
cd $NWJS
gclient config --name=src https://github.com/nwjs/chromium.src.git@origin/nw-v0.26.5
```

Put `NWJS` environment variable in your ~/.bashrc or ~/.zshrc.

Generally if you are not interested in running Chromium tests, you don't have to sync the test cases and reference builds, which saves you lot of time. Open the `.gclient` file you just created and replace `custom_deps` section with followings:

```python
"custom_deps" : {
    "src/third_party/WebKit/LayoutTests": None,
    "src/chrome_frame/tools/test/reference_build/chrome": None,
    "src/chrome_frame/tools/test/reference_build/chrome_win": None,
    "src/chrome/tools/test/reference_build/chrome": None,
    "src/chrome/tools/test/reference_build/chrome_linux": None,
    "src/chrome/tools/test/reference_build/chrome_mac": None,
    "src/chrome/tools/test/reference_build/chrome_win": None,
}
```

**Step 2.** Manually clone and checkout correct branches for following repositories:

| path | repo |
|:---- |:---- |
| src/content/nw | https://github.com/nwjs/nw.js |
| src/third_party/node | https://github.com/nwjs/node |
| src/v8 | https://github.com/nwjs/v8 |


```bash
mkdir -p $NWJS/src/content/nw
mkdir -p $NWJS/src/third_party/node-nw
mkdir -p $NWJS/src/v8
git clone https://github.com/nwjs/nw.js $NWJS/src/content/nw
git clone https://github.com/nwjs/node $NWJS/src/third_party/node-nw
git clone https://github.com/nwjs/v8 $NWJS/src/v8
cd $NWJS/src/content/nw
git checkout nw-v0.26.5
cd $NWJS/src/third_party/node-nw
git checkout nw-v0.26.5
cd $NWJS/src/v8
git checkout nw-v0.26.5
```

**Step 3.** Export cross-compilation environment variables and synchronize the projects. To enable proprietary codecs set `ffmpeg_branding` to `Chrome` when you configure GN!

```bash
cd $NWJS/src
export GYP_CROSSCOMPILE="1"
export GYP_DEFINES="target_arch=arm arm_float_abi=hard nwjs_sdk=1 disable_nacl=0"
export GN_ARGS="is_debug=false is_component_ffmpeg=true enable_nacl=true target_cpu=\"arm\" ffmpeg_branding=\"Chrome\""

export GYP_CHROMIUM_NO_ACTION=1
gclient sync --reset --with_branch_heads --nohooks
```

This usually downloads 20G+ from remote repositories.

**Step 4.** Install all the compiler and library dependencies:
```bash
./build/install-build-deps.sh --arm
```

**Step 5.** Apply some ARMv7 specific patches:
```bash

cd $NWJS/src
patch -Np1 < /path/to/chromium.patch
cd $NWJS/src/content/nw/
patch -Np1 < /path/to/nw.patch
cd $NWJS/src/third_party/node-nw/
patch -Np1 < /path/to/node.patch

```

**Step 6.** Setup environment variables and generate ninja build files with GN for Chromium:
```bash
cd $NWJS/src
gclient runhooks
gn gen out_gn_arm/nw --args="$GN_ARGS"
export GYP_CHROMIUM_NO_ACTION=0
python build/gyp_chromium -Goutput_dir=out_gn_arm -I third_party/node-nw/common.gypi third_party/node-nw/node.gyp
```

### Build

Build NW.js and Node:
```bash
ninja -C out_gn_arm/nw nwjs
ninja -C out_gn_arm/Release node
ninja -C out_gn_arm/nw copy_node
ninja -C out_gn_arm/nw dump
ninja -C out_gn_arm/nw dist
```

This process can take few hours depending on your system configuration.

When the compilation is done you should find your artifacts under `$NWJS/src/out_gn_arm/nw/dist/`.

[Xubuntu 16.04.1 LTS (Xenial Xerus)]: http://cdimage.ubuntu.com/xubuntu/releases/xenial/release/xubuntu-16.04.1-desktop-amd64.iso
[Building NW.js]: http://docs.nwjs.io/en/latest/For%20Developers/Building%20NW.js/
[depot_tools package]: https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up