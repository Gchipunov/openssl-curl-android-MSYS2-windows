# openssl-curl-android

Compile openssl and curl for Android

## Prerequisites

Make sure you have `Android NDK` installed.

You may also need to install `autoconf` and `libtool` toolchains as well as build essentials.
clone repo beefore download with command prompt
enter MSYS2
## Download MSYS2 and configure enviroment might need to download other pacman
```
in MSYS2 terminal:
echo $MSYSTEM


export NDK=/D/android-sdk/ndk/23.1.7779620 # e.g. D:/android-sdk/ndk/23.0.7599858
export HOST_TAG=windows-x86_64 # e.g. darwin-x86_64, see https://developer.android.com/ndk/guides/other_build_systems#overview
export MIN_SDK_VERSION=21 # or any version you want

pacman -S mingw-w64-ucrt-x86_64-python mingw-w64-ucrt-x86_64-python-pip
#pacman -S mingw-w64-x86_64-python mingw-w64-x86_64-python-pip
#?? pip install diskutils

python -v
cd /d/android-sdk/ndk/23.1.7779620/build/tools
python make_standalone_toolchain.py --api 21 --arch arm64 --install-dir /d/android-toolchain2/arm64
```
output something like:
k10@DESKTOP-DVP2O8B UCRT64 /d/android-sdk/ndk/23.1.7779620/build/tools
$ python make_standalone_toolchain.py --api 21 --arch arm64 --install-dir /d/android-toolchain2/arm64
WARNING:__main__:make_standalone_toolchain.py is no longer necessary. The
%NDK%/toolchains/llvm/prebuilt/windows-x86_64/bin directory contains target-specific scripts that perform
the same task. For example, instead of:

    C:\>python %NDK%/build/tools/make_standalone_toolchain.py \
        --arch arm64 --api 21 --install-dir toolchain
    C:\>toolchain/bin/clang++ src.cpp

Instead use:

    C:\>%NDK%/toolchains/llvm/prebuilt/windows-x86_64/bin/aarch64-linux-android21-clang++ src.cpp

open D:\android-toolchain2\arm64\bin\aarch64-linux-android21-clang change clang.exe to clang120.exe or what ever is there
#!/bin/bash
if [ "$1" != "-cc1" ]; then
    `dirname $0`/clang.exe --target=aarch64-linux-android21 "$@"
else
    # Target is already an argument.
    `dirname $0`/clang120.exe "$@"
fi



```

pacman -S autoconf automake
pacman -S libtool

chmod +x ./build.sh
./build.sh


```
If you do not want to compile them yourself, you can download pre-compiled static libraries from [releases](https://github.com/robertying/openssl-curl-android/releases). They are in `build.tar.gz`.

Doing your own compilation is recommended, since the pre-compiled binary can become outdated soon.

Checkout newer versions in git submodules to compile newer versions of the libraries. For example, to build `OpenSSL_1_1_1l` and `curl-7_78_0`:

```bash
cd openssl
git fetch
git checkout OpenSSL_1_1_1l
cd ..

cd curl
git fetch
git checkout curl-7_78_0
cd ..
```

## Usage

```bash
git clone https://github.com/robertying/openssl-curl-android.git
cd openssl-curl-android
git submodule update --init --recursive

export NDK=your_android_ndk_root_here # e.g. $HOME/Library/Android/sdk/ndk/23.0.7599858
export HOST_TAG=see_this_table_for_info # e.g. darwin-x86_64, see https://developer.android.com/ndk/guides/other_build_systems#overview
export MIN_SDK_VERSION=23 # or any version you want

chmod +x ./build.sh
./build.sh
```

All compiled libs are located in `build/openssl` and `build/curl` directory.

Use NDK to link those libs, part of `Android.mk` example:

```makefile
include $(CLEAR_VARS)
LOCAL_MODULE := curl
LOCAL_SRC_FILES := build/curl/$(TARGET_ARCH_ABI)/libcurl.a
include $(PREBUILT_STATIC_LIBRARY)
```

## Options

- Change scripts' configure arguments to meet your own needs.

- For now, using TLS (https) in Android would throw `peer verification failed`.

  Please explicitly set `curl_easy_setopt(curl, CURLOPT_CAINFO, CA_BUNDLE_PATH);` where `CA_BUNDLE_PATH` is your ca bundle path in the device storage.

  You can download and copy [cacert.pem](https://curl.haxx.se/docs/caextract.html) to Android assets or the device internal storage to get TLS working for libcurl.

## Working Examples

- See this minimal example which calls `curl` from Android app, using `JNI` to use `libcurl`: [AndroidCurlExample](https://github.com/robertying/AndroidCurlExample). It includes `Android.mk` setup and `JNI` configurations.

- Checkout this more complex [repo](https://github.com/robertying/CampusNet-Android/blob/master/app/src/main/cpp/jni) to see how to integrate other compiled static libraries into an existing Android project, including `Android.mk` setup and `JNI` configurations.
