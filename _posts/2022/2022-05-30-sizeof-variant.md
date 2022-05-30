---
title: sizeof(std::variant)
---

I was debugging a memory problem with the [SOM training](https://en.wikipedia.org/wiki/Self-organizing_map)
of the PHZ pipeline. Even if the input file was just around 100 MiB,
the memory consumption would grow up to 4 GiB without any evident explanation.

It turns out that [Alexandria's Table](https://github.com/astrorama/Alexandria/blob/master/Table/Table/Row.h#L68)
class is just too flexible. It can read POD as `float`, `double`, `int`, but also
more complex types as `std::vector<int>` or `NdArray<int>`.
The latter is similar to numpy's `ndarray`, so it has to book-keep more
information that a plain `std::vector`: i.e., shape, strides, underlying
container, etc.

`Table::Row` does this using a `boost::variant` with all the supported types,
which is all fine... except that the variant will keep as much memory as the
biggest type (like a `union`), plus a type flag, plus any padding that may be
required.

`sizeof(NdArray<int>)` was 112 bytes or so, blowing up the memory required for
each individual cell.

To reduce the memory required by an `NdArray` I changed this:

```cpp
class NdArray {
private:
  size_t                   m_offset;
  std::vector<size_t>      m_shape, m_stride_size;
  std::vector<std::string> m_attr_names;
  size_t                   m_size, m_total_stride;
  std::shared_ptr<ContainerInterface> m_container;
};
```

By sort-of a pimpl idiom:

```cpp
class NdArray {
private:
  struct Details {
    size_t                   m_offset;
    std::vector<size_t>      m_shape, m_stride_size;
    std::vector<std::string> m_attr_names;
    size_t                   m_size, m_total_stride;
    std::shared_ptr<ContainerInterface> m_container;
  };
  std::unique_ptr<Details> m_details_ptr;
};
```

Now `sizeof(NdArray)` is just 8 bytes.
Sure, it complicates the constructors and require some indirection,
but the memory used when reading a catalog is greatly reduced.
