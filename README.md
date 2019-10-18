# Shrink LinAlg
This is a sample Dockerfile on how to optimize the disk space consumption from
NumPy, SciPy, Pandas, and Matplotlib.

An in-depth write up is also on my [blog post on Medium](https://medium.com/@szelenka/how-to-shrink-numpy-scipy-pandas-and-matplotlib-for-your-data-product-4ec8d7e86ee4).

## PIP Install Options
When installing through PIP within a Docker container, there’s no point in keeping around the cache. Lets add some flags to instruct PIP how to function.

— no-cache-dir
PIP uses caching to prevent duplicate HTTP requests to pull modules from the repository before installing on the local system. When running inside a Docker image, there’s no need to preserve this cache, so disable it with this flag.

— compile
Compile Python source files to bytecode. When running inside a Docker image, it’s highly unlikely that you’ll need to debug a mature installed module (such as NumPy, SciPy, Pandas, or Matplotlib).

— global-option=build_ext
To inform the C compiler we want to add additional flags during compile and link, we need to set these additional “global-option” flags.

## C Compiler Flags
Python has a wrapper for C-Extension called Cython, which enables developers to write C code in a Python-like syntax. The advantage of this, is that it allows for a lot of the optimization of C, but with the ease of writing Python. NumPy, SciPy, and Pandas leverage Cython a lot! Matplotlib appears to contain some Cython as well, but to a much lesser extent.

These compiler flags are passed to the GNU complier installed within your Dockerfile. To cut to the chase, we’re going to investigate only a handful of them:

Disable debug statements (-g0)
Because we’re sticking our data product into a Docker image, it’s highly unlikely we’ll be doing any real-time debugging of the Cython code from NumPy, SciPy, Pandas, or Matplotlib.

Remove symbol files (-Wl, — strip-all)
If we’re never going to debug the build of these packages within the Docker image, there’s no sense in keeping around the symbol files required for debugging either.

Optimize for nearly all supported optimizations that do not involve a space-speed tradeoff (-O2) or Optimize for disk space (-Os)
The challenge with these optimization flags is that while it can increase/decrease the disk size, it also manipulates the run-time performance of the compiled binary! Without explicitly testing the impact it has on our data product, these could be 
risky (but at the same time, they’re very efficient).

Location of header files (-I/usr/include:/usr/local/include)
Be explicit in telling GCC where to look for the header files needed to compile the Cython modules.

Location of library files (-L/usr/lib:/usr/local/lib)
Be explicit in telling GCC where to look for the library files needed to compile the Cython modules.

But what’s the impact setting these CFLAG during install via PIP?

The lack of debug information may cause experienced developers to raise caution, but if you really need them at a later date, you can always re-build the Docker image with the flag to reproduce any stacktrace or core dump happening. The larger concern is for exceptions which aren’t reproducible .. which would not be diagnosable without the symbol files. But again, nobody does that in a Docker image.
