# CrossGuid [![Build Status](https://travis-ci.com/goto-bus-stop/crossguid.svg?branch=master)](https://travis-ci.com/goto-bus-stop/crossguid)

CrossGuid is a minimal, cross platform, C++ GUID library. It uses the best
native GUID/UUID generator on the given platform and has a generic class for
parsing, stringifying, and comparing IDs. The guid generation technique is
determined by your platform:

## Linux

On linux you can use `libuuid` which is pretty standard. On distros like Ubuntu
it is available by default but to use it you need the header files so you have
to do:

    sudo apt-get install uuid-dev

## Mac/iOS

On Mac or iOS you can use `CFUUIDCreate` from `CoreFoundation`. Since it's a
plain C function you don't even need to compile as Objective-C++.

## Windows

On Windows we use the built-in function `CoCreateGuid`. CMake can
generate a Visual Studio project if that's your thing.

## Android

The Android version uses a handle to a `JNIEnv` object to invoke the
`randomUUID()` function on `java.util.UUID` from C++. The Android specific code
is all in the `android/` subdirectory. If you have an emulator already running,
then you can run the `android.sh` script in the root directory. It has the
following requirements:

- Android emulator is already running (or you have physical device connected).
- You're using bash.
- adb is in your path.
- You have an Android sdk setup including `ANDROID_HOME` environment variable.

## Compiling

Do the normal cmake thing:

```
mkdir build
cd build
cmake ..
make install
```

## Running tests

After compiling as described above you should get two files: `libcrossguid.a` (the
static library) and `crossguid-test` (the test runner). So to run the tests do:

```
./crossguid-test
```

## Basic usage

### Creating guids

Create a new random guid:

```cpp
#include <crossguid/guid.hpp>

auto g = xg::newGuid();
```

**NOTE:** On Android you need to call `xg::initJni(JNIEnv *)` first so that it
is possible for `xg::newGuid()` to call back into java libraries. `initJni`
only needs to be called once when the process starts.

Create a new zero guid:

```cpp
xg::Guid g;
```

Create from a string:

```cpp
xg::Guid g("c405c66c-ccbb-4ffd-9b62-c286c0fd7a3b");
```

All `xg::Guid` constructors are constexpr so you can initialize GUIDs at compile time.
```cpp
static constexpr auto someGuid = xg::Guid("46744ec1-5d05-45ae-a356-776fbd04d103");
```

### Checking validity

If you have some string value and you need to check whether it is a valid guid
then you can simply attempt to construct the guid:

```cpp
xg::Guid g("bad-guid-string");
if (!g.isValid())
{
	// do stuff
}
```

If the guid string is not valid then all bytes are set to zero and `isValid()`
returns `false`.

### Converting guid to string

First of all, there is normally no reason to convert a guid to a string except
for in debugging or when serializing for API calls or whatever. You should
definitely avoid storing guids as strings or using strings for any
computations. If you do need to convert a guid to a string, then you can
utilize strings because the `<<` operator is overloaded. To print a guid to
`std::cout`:

```cpp
void doGuidStuff() {
  auto myGuid = xg::newGuid();
  std::cout << "Here is a guid: " << myGuid << std::endl;
}
```

Or to convert a guid to a `std::string`:

```cpp
std::string guidStr = xg::newGuid().str();
```

### Creating a guid from raw bytes

It's unlikely that you will need this, but this is done within the library
internally to construct a `Guid` object from the raw data given by the system's
built-in guid generation function.

```cpp
Guid(std::array<unsigned char, 16> &bytes);
```

### Comparing guids

`==` and `!=` are implemented, so the following works as expected:

```cpp
void doGuidStuff() {
  auto guid1 = xg::newGuid();
  auto guid2 = xg::newGuid();

  auto guidsAreEqual = guid1 == guid2;
  auto guidsAreNotEqual = guid1 != guid2;
}
```

### Hashing guids

Guids can be used directly in containers requireing `std::hash` such as `std::map`, `std::unordered_map` etc.

## License

The MIT License (MIT)

Copyright (c) 2014 Graeme Hill (http://graemehill.ca)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
