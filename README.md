# HDF5 interface for the Julia language

[![Build Status](https://travis-ci.org/JuliaIO/HDF5.jl.svg?branch=master)](https://travis-ci.org/JuliaIO/HDF5.jl) [![Build status](https://ci.appveyor.com/api/projects/status/c00htf9yte5swb1p/branch/master?svg=true)](https://ci.appveyor.com/project/timholy/hdf5-jl-1fndw/branch/master)
[![Coverage Status](https://coveralls.io/repos/JuliaIO/HDF5.jl/badge.png?branch=master)](https://coveralls.io/r/JuliaIO/HDF5.jl?branch=master)

[HDF5][HDF5] is a file format and library for storing and accessing
data, commonly used for scientific data. HDF5 files can be created and
read by numerous [programming
languages](http://www.hdfgroup.org/tools5desc.html).  This package
provides an interface to the HDF5 library for the
[Julia][Julia] language.

## Julia data (\*.jld) and Matlab (\*.mat) files

The core HDF5 functionality is the foundation for two special-purpose
packages, used to read and write HDF5 files with specific formatting
conventions. The first is the
[JLD](https://github.com/JuliaIO/JLD.jl) ("Julia data") package,
which implements a generic mechanism for reading and writing Julia
variables. While one can use "plain" HDF5 for this purpose, the
advantage of the JLD package is that it preserves the exact type
information of each variable.

The other functionality provided through HDF5 is the ability to read
and write Matlab \*.mat files saved as "-v7.3". The Matlab-specific
portions have been moved to Simon Kornblith's
[MAT.jl](https://github.com/simonster/MAT.jl) package.

## Installation

Within Julia, use the package manager:
```julia
Pkg.add("HDF5")
```

You also need to have the HDF5 library installed on your
system (version 1.8 or higher is required), but **for most users
no additional steps should be required; the HDF5 library should be
installed for you automatically when you add the package.**

If you have to install the HDF5 library manually, here are some examples of
how to do it:

- Debian/(K)Ubuntu: `apt-get -u install hdf5-tools`
- OSX: `brew tap homebrew/science; brew install hdf5` (using [Homebrew](http://brew.sh))
- Windows: It is highly recommended that you use the HDF5 library
  fetched by this package. Other HDF5 binaries may be compiled against
  a different C runtime from the Julia binary, which will cause
  Julia to crash when freeing memory allocated by libhdf5.

If you've installed the library but discover that Julia is not finding
it, you can add the path to Julia's `Libdl.DL_LOAD_PATH` variable, e.g.,
```julia
push!(Libdl.DL_LOAD_PATH, "/opt/local/lib")
Pkg.build("HDF5")
```
Inserting this command into your `.juliarc.jl` file will cause this to
happen automatically each time you start Julia.

If you're on Linux but you do not have root privileges on your machine (and
you can't persuade the sysadmin to install the libraries for you), you can [download](http://www.hdfgroup.org/HDF5/release/obtain5.html) the
binaries and place them somewhere in your home directory. To use HDF5,
you'll have to start julia as
```
LD_LIBRARY_PATH=/path/to/hdf5/libs julia
```
You can set up an alias so this happens for you automatically each time
you start julia.

## Quickstart

Begin your code with

```julia
using HDF5
```

To read and write a variable to a file, one approach is to use the filename:
```julia
A = collect(reshape(1:120, 15, 8))
h5write("/tmp/test2.h5", "mygroup2/A", A)
data = h5read("/tmp/test2.h5", "mygroup2/A", (2:3:15, 3:5))
```
where the last line reads back just `A[2:3:15, 3:5]` from the dataset.

More fine-grained control can be obtained using functional syntax:

```julia
h5open("mydata.h5", "w") do file
    write(file, "A", A)  # alternatively, say "@write file A"
end

c = h5open("mydata.h5", "r") do file
    read(file, "A")
end
```
This allows you to add variables as they are generated to an open HDF5 file.
You don't have to use the `do` syntax (`file = h5open("mydata.h5", "w")` works
just fine), but an advantage is that it will automatically close the file (`close(file)`)
for you, even in cases of error.

Julia's high-level wrapper, providing a dictionary-like interface, may
also be of interest:

```julia
using HDF5

h5open("test.h5", "w") do file
    g = g_create(file, "mygroup") # create a group
    g["dset1"] = 3.2              # create a scalar dataset inside the group
    attrs(g)["Description"] = "This group contains only a single dataset" # an attribute
end
```

Convenience functions for attributes attached to datasets are also provided:

```julia
  A = Vector{Int}(1:10)
  h5write("bar.h5", "foo", A)
  h5writeattr("bar.h5", "foo", Dict("c"=>"value for metadata parameter c","d"=>"metadata d"))
  h5readattr("bar.h5", "foo")
```


## Specific file formats

There is no conflict in having multiple modules (HDF5, [JLD](https://github.com/JuliaIO/JLD.jl), and
[MAT](https://github.com/simonster/MAT.jl)) available simultaneously;
the formatting of the file is determined by the open command.

## Complete documentation

The HDF5 API is much more extensive than suggested by this brief
introduction.  More complete documentation is found in the
[`doc`](doc/) directory.

The [`test`](test/) directory contains a number of test scripts that also
demonstrate usage.

## Credits

- Konrad Hinsen initiated Julia's support for HDF5

- Tim Holy and Simon Kornblith (co-maintainers and primary authors)

- Tom Short contributed code and ideas to the dictionary-like
  interface

- Blake Johnson made several improvements, such as support for
  iterating over attributes

- Isaiah Norton and Elliot Saba improved installation on Windows and OSX

- Steve Johnson contributed the `do` syntax and Blosc compression

- Mike Nolta and Jameson Nash contributed code or suggestions for
  improving the handling of HDF5's constants

- Thanks also to the users who have reported bugs and tested fixes


[Julia]: http://julialang.org "Julia"
[HDF5]: http://www.hdfgroup.org/HDF5/ "HDF5"
