# MKL.NET

<p>
<a href="https://github.com/AnthonyLloyd/MKL.NET/actions"><img src="https://github.com/AnthonyLloyd/MKL.NET/workflows/CI/badge.svg?branch=master"></a>
</p>

A simple cross platform .NET API for Intel MKL.

Exposing functions from MKL keeping the syntax as close to the
[c developer reference](https://software.intel.com/content/www/us/en/develop/documentation/mkl-developer-reference-c/top.html) as possible.

Reference the MKL.NET package and required runtime packages and use the static MKL functions.
The correct native libraries will be included and loaded at runtime.

<table>
<tr><td>MKL.NET</td><td><a href="https://www.nuget.org/packages/MKL.NET"><img src="https://buildstats.info/nuget/MKL.NET?includePreReleases=true" ></a></td></tr>
<tr><td>runtimes:</td></tr>
<tr><td>MKL.NET.win-x64</td><td><a href="https://www.nuget.org/packages/MKL.NET.win-x64"><img src="https://buildstats.info/nuget/MKL.NET.win-x64?includePreReleases=true" ></a></td></tr>
<tr><td>MKL.NET.win-x86</td><td><a href="https://www.nuget.org/packages/MKL.NET.win-x86"><img src="https://buildstats.info/nuget/MKL.NET.win-x86?includePreReleases=true" ></a></td></tr>
<tr><td>MKL.NET.linux-x64</td><td><a href="https://www.nuget.org/packages/MKL.NET.linux-x64"><img src="https://buildstats.info/nuget/MKL.NET.linux-x64?includePreReleases=true" ></a></td></tr>
<tr><td>MKL.NET.linux-x86</td><td><a href="https://www.nuget.org/packages/MKL.NET.linux-x86"><img src="https://buildstats.info/nuget/MKL.NET.linux-x86?includePreReleases=true" ></a></td></tr>
<tr><td>MKL.NET.osx-x64</td><td><a href="https://www.nuget.org/packages/MKL.NET.osx-x64"><img src="https://buildstats.info/nuget/MKL.NET.osx-x64?includePreReleases=true" ></a></td></tr>
<tr><td>libraries:</td></tr>
<tr><td>MKL.NET.Matrix</td><td><a href="https://www.nuget.org/packages/MKL.NET.Matrix"><img src="https://buildstats.info/nuget/MKL.NET.Matrix?includePreReleases=true" ></td></tr>
<tr><td>MKL.NET.Optimization</td><td><a href="https://www.nuget.org/packages/MKL.NET.Optimization"><img src="https://buildstats.info/nuget/MKL.NET.Optimization?includePreReleases=true" ></td></tr>
<tr><td>MKL.NET.Statistics</td><td><a href="https://www.nuget.org/packages/MKL.NET.Statistics"><img src="https://buildstats.info/nuget/MKL.NET.Statistics?includePreReleases=true" ></td></tr>
</table>

## Rationale

- Use freely available Intel MKL packages repackaged to work for each runtime.
- The MKL.NET API is just a thin .NET wrapper around the native API keeping the syntax as close as possible.
- The project is well defined with no business logic and could benefit from external input.
- Cross platform testing is easy and free using Github actions.
- MKL.NET native packages can just be referenced for needed runtimes at library or application level.

## MKL.NET.Matrix

- Performance and memory optimised matrix algebra library.
- Matrix expressions are optimised to perform intermediate calculations inplace and reuse memory.
- Operations such as scale, transpose, +, * are combined into single MKL calls.
- Intermediate matrices are disposed (or reused) automatically.
- ArrayPool underlying memory model using IDisposable and Finalizers.
- Uses the Pinned Object Heap for net5.0.
- All these combined result in it being much faster than other matrix libraries.

The following example only results in one new matrix r (using ArrayPool) without mutating inputs.
```csharp
public static matrix Example(matrix ma, matrix mb, vector va, vector vb)
{
    using matrix r = 0.5 * Matrix.Abs(1.0 - ma) * mb.T + Math.PI * va.T * Vector.Sin(vb);
    ...
}
```

Example statistics matrix function:
```csharp
public static (vector, matrix) MeanAndCovariance(matrix samples, vector weights)
{
    if (samples.Rows != weights.Length) ThrowHelper.ThrowIncorrectDimensionsForOperation();
    var mean = new vector(samples.Cols);
    var cov = new matrix(samples.Cols, samples.Cols);
    var task = Vsl.SSNewTask(samples.Cols, samples.Rows, VslStorage.ROWS, samples.Array, weights.Array);
    ThrowHelper.Check(Vsl.SSEditCovCor(task, mean.Array, cov.Array, VslFormat.FULL, null, VslFormat.FULL));
    ThrowHelper.Check(Vsl.SSCompute(task, VslEstimate.COV, VslMethod.FAST));
    ThrowHelper.Check(Vsl.SSDeleteTask(task));
    return (mean, cov);
}
```

Note: arrays need to be pinned across all MKL function calls when there are multiple as above as MKL stores native pointers and the arrays could be moved between calls.
MKL.NET handles pinning automatically, unpinning when the task is deleted.
This is a common seen bug when using MKL directly from .NET which causes occasional crashes.

## MKL.NET.Optimization

Simple and high performance optimization and root finding library loosely based on the [scipy.optimize](https://docs.scipy.org/doc/scipy/reference/optimize.html) API.

The aim is to include the latest algorithms such as Toms748, robustly tested with [CsCheck](https://github.com/AnthonyLloyd/CsCheck).
Full use of MKL.NET will be made to improve performance. Algorithms will be performance tested and default to the best for given inputs.

- Root - root finding algorithms. Default algorithm has 20% fewer function calls than Brent, Toms748, Newton and Halley.  
- Calculus - derivative and integral numeric calculations and check to any precision using Richardson extrapolation.

WIP minimize scalar

## MKL.NET.Statistics

Simple and high performance statistics functions loosely based on the [scipy.stats](https://docs.scipy.org/doc/scipy/reference/stats.html) API.

WIP ...