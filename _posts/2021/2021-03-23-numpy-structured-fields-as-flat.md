---
title: Accessing fields from a numpy structured array as a "regular" array
---

The whole 2020 is gone, and I hadn't written a single entry!
Well, it is not like anything happened during 2020, did it? 🤔

Anyhow, so the subject of this entry is: how to access a set of fields
on a [numpy](https://numpy.org/) structured array as if it were a simple,
plain, array?

A concrete example: I have some code that reads from a FITS catalog a set
of [fluxes measured on different bands](https://en.wikipedia.org/wiki/Photometry_(astronomy).
For instance, `ugriz`. That's read into a fits binary table in Python, which
wraps pretty much a [numpy structured array](https://numpy.org/doc/stable/user/basics.rec.html)
with a field per column.

Of course, the table has mixed types: floating point for the photometry,
integers for the object ID, Boolean or integers for flagging, etc.

However, there is some code around that does not work with these kind of
data, and expects to received an unstructured array instead, sometimes
with two axes (number of rows x number of bands), sometimes with three
(number of rows x number of bands x [value,error]).

In general, just copying the data over a unstructured array might be
just fine. For instance

```python
data =  np.zeros((len(phot_table), len(filter_list), 2), dtype=np.float32)
for i, name in enumerate(filter_list):
    data[:, i, 0] = phot_table[name]
```

But sometimes that implies having the same data twice in memory, and the
size is non negligible.

This would also happen when doing the reverse: writing a catalog from a
unstructured array stored in memory.

The last case was giving trouble, in fact. We would compute a set of
"uniform photometry"[^1] for the target catalog, and write
the results into a FITS catalog. For a moment, generating the output table
would create a copy of the data just for the purpose of serialization,
increasing the peak memory footprint by a ridiculous and wasteful amount[^2].

The obvious thought would be to allocate the output buffer first, and use it
for computations too. But the output will be a structured array, and we need
an unstructured one.

My Google-fu and StackOverflow search failed me miserably, and could only find
how to get a view of a subset of the fields, but that is still a structured
array.

However, one can manually create arrays with a provided buffer that can point
to some other array, with any arbitrary casting. And a `struct` with four
fields is, from the memory layout point of view, pretty much indistinguishable
from an array of size 4![^3]

So, as long as the fields we want to access are consecutive in memory, *and*
they have the same type, we can create a custom array. For instance, if we
have the fields we want in `fields`:

```python
view = np.ndarray(data.shape, dtype=(dtype, len(fields)), buffer=data,
                  offset=data.dtype.fields[fields[0]][1], strides=data.strides)
```

**Beware!** If the fields happen *not* to have the same type, you may get
garbage on some entries, since the memory is re-interpreted as the new type.
Also, if the fields happen not to be consecutive, you will be accessing
the wrong data.

These preconditions can be tested as follows:

```python
selected = [data.dtype.fields[c] for c in fields]
dtypes = [f[0] for f in selected]
offsets = np.array([f[1] for f in selected])
sizes = np.array([data.dtype[f].itemsize for f in fields])
if len(set(dtypes)) > 1:
    raise TypeError('All fields must have the same type')

# The offset of the i field + its size must correspond to the offset of the i+1 field
consecutive = (offsets[:-1] + sizes[:-1] == offsets[1:]).all()
if not consecutive:
    raise IndexError('All fields must be consecutive')
```

If you feel paranoid, you can double check that, indeed, the data is shared

```python
assert np.may_share_memory(view, data)
```

At the end, `view` is an array of a given type, with two axes: the first
corresponds to the number of entries, and the second to the number of
selected fields.

```python
view[:, 0:2] *= 5
# Equivalent to
data[['field0', 'field1']] *= 5
```


[^1]: eli5: comparable

[^2]: Even when most of the memory we use can be safely swapped out, it still
      exhaust the limit set by the job manager on the cluster. Reserving more
      memory implies reserving more CPUs (we get 4 GiB per core)
      that will just sit there idle.

[^3]: Well, it depends on the padding and alignment.
