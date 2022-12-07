---
title: Trying std::format
---

I am trying to test [`std::format`](https://en.cppreference.com/w/cpp/utility/format/format),
but, unfortunately, it is not fully available in `gcc` nor `clang`.
I know, I know, I could use [`fmt`](https://fmt.dev/latest/index.html) instead, but I need `std::format`
because they are not identical.

For the record, what I have done is:

1. Get the latest llvm sources
2. Build `libc++`, and company, enabling the experimental features [^1]
3. Install it under `/opt/clang/16/`

```bash
git clone https://github.com/llvm/llvm-project.git --depth 1
cd llvm-project
cmake -G Ninja -S llvm -B build \
    -DLLVM_ENABLE_PROJECTS="clang" \
    -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind" \
    -DLIBCXX_ENABLE_INCOMPLETE_FEATURES=ON \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/clang/16/
ninja -C build runtimes
ninja -C build install-cxx install-cxxabi install-unwind
```

Once that is done, I configure the test project with the following flags:

```bash
cmake \
    -DCMAKE_CXX_FLAGS="-nostdinc++ -nostdlib++ -fexperimental-library \
        -isystem /opt/clang/16/include/c++/v1 \
        -isystem /opt/clang/16/include/x86_64-unknown-linux-gnu/c++/v1" \
    -DCMAKE_EXE_LINKER_FLAGS="-L /opt/clang/16/lib/x86_64-unknown-linux-gnu\
        -Wl,-rpath,/opt/clang/16/lib/x86_64-unknown-linux-gnu\
        -lc++ -fuse-ld=lld -lc++experimental"
```

Note that the system default linker (`bfd`) didn't work, so I had to use `lld` instead.
I am (still) running an Ubuntu 20.04 at work.

With that, `#include <format>` works 😄

[^1]: [Building libc++](https://libcxx.llvm.org/BuildingLibcxx.html)
