There are a couple changes that you have to make to the cmake files in the fprime folder. Eventually we should create a fork of the repository but for now i'm just going to make the changes locally.

### fprime/cmake/settings.cmake
At the very bottom of this file find this block of code

```cmake
if (BUILD_TESTING)

add_compile_options(-g -DBUILD_UT -DPROTECTED=public -DPRIVATE=public -DSTATIC=)

endif()
```
and add -lrt to the end of it as so

```cmake
if (BUILD_TESTING)

add_compile_options(-g -DBUILD_UT -DPROTECTED=public -DPRIVATE=public -DSTATIC= -lrt)

endif()
```

### fprime/cmake/toolchain/arm-hf-linux.cmake
Anywhere in this file add the following lines
```cmake
set(CMAKE_CXX_FLAGS_INIT "-lrt")

set(CMAKE_CXX_EXE_FLAGS_INIT "-lrt")
```

Once you've added these changes it should run when you compile using `fprime-util build arm-hf-linux`