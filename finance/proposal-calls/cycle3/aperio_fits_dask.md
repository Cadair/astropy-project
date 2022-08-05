### Title
Improve FITS compressed image support and Dask integration in `io.fits`.

### Project Team
Stuart Mumford
Thomas Robitaille
Drew Leonard

### Project Description

This project aims to increase performance of `io.fits` especially around large
compressed image data.

#### Compressed Image Performance

Currently when using tiled FITS compression the Astropy implementation loads
all the tiles into RAM and decompresses them all.
This is highly CPU and memory inefficient, and is one of the reasons Astropy is
significantly slower at loading these types of files than the `cfitsio` package
(which uses the same C library as Astropy for this currently).

For example we can compare the loading times of `io.fits` vs `cfitsio`:

Loading a whole array with astropy:

```
In [44]: %timeit fits.getdata("VBI_L1_00656282_2018_05_11T14_25_05_466665_I.fits", hdu=1)
183 ms ± 248 µs per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

with cfitsio:
```
In [45]: %timeit hdu[1][:, :]
131 ms ± 182 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

loading one or more individual tiles with `cfitsio`:
```
In [48]: %timeit hdu[1][0, 0]
21.6 µs ± 191 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)

In [49]: %timeit hdu[1][0, :]
22.3 µs ± 78.2 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)

In [50]: %timeit hdu[1][:10, :]
235 µs ± 718 ns per loop (mean ± std. dev. of 7 runs, 1,000 loops each)
```

note that doing the same operations with astropy currently loads the whole file
so takes the same amount of time as the first `getdata` call. This means that
loading a single tile with astropy is approximately 8300x slower than with
`cfitsio`!!

Issue [#3895](https://github.com/astropy/astropy/issues/3895) has been open
since 2015 and describes a set of deep improvements for the compressed image
support in `io.fits`. This includes implementing the compression natively in
Python rather than using the `fitsio` C library to do it. This will have the
side effect of significantly reducing the compile time complexity of Astropy, as
this is the only part of the code which uses `fitsio`.

Some relevant issues:

* https://github.com/astropy/astropy/issues/3895
* https://github.com/astropy/astropy/issues/11633
* https://github.com/astropy/astropy/issues/9238


#### Dask Integration with reading FITS files

While this is one of the largest performance issues in `io.fits`, this project
is also proposing to increase the integration of Dask with `io.fits` which will
enable significant performance improvements when using FITS files with
distributed compute.

### Approximate Budget

Money in return for the sacrifice of sanity required to rework large chunks of
`io.fits`. /s