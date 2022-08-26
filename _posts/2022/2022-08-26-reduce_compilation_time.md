I watched the episode [Analyzing and Improving Build Times](https://youtu.be/Iybb9wnpF00)
from C++ Weekly the other day.

It looked exciting, so I decided to give it a shot and try to
profile the build time of [PhosphorosCore](https://github.com/astrorama/PhosphorosCore)
using `clang++` and its flag `-ftime-trace`. It did work, but I got a single
`json` file per translation unit. Still helpful, but I was looking for more of
a high-level overview.

A project does just that: [ClangBuildAnalyzer](https://github.com/aras-p/ClangBuildAnalyzer).
It can aggregate all those `json` files into a single file and obtain a summary.
i.e.:

```bash
ClangBuildAnalyzer --all build.x86_64-fc35-clang120-dbg FullCapture.bin
ClangBuildAnalyzer --analyze FullCapture.bin > Report.txt
```

An extract from the output:

```
173540 ms: /home/aalvarez/Work/Projects/PhosphorosCore/PhosphorosCore/PhzDataModel/PhzDataModel/PhotometryGrid.h (included 122 times, avg 1422 ms), included via:
  CheckLuminosityParameter.cpp.o CheckLuminosityParameter.h  (2912 ms)
  BestModel.cpp.o BestModel.h  (2820 ms)
  PhotometryGrid.cpp.o  (2692 ms)
  PhysicalParameter.cpp.o PhysicalParameter.h  (2656 ms)
  PhotometryGridCreator_test.cpp.o PhotometryGridCreator.h  (2484 ms)
  CatalogHandler.cpp.o CatalogHandler.h  (2461 ms)
  ...

161148 ms: /home/aalvarez/Work/Projects/Alexandria/2.26.0/InstallArea/x86_64-fc35-clang120-o2g/include/GridContainer/serialize.h (included 135 times, avg 1193 ms), included via:
  ModelDatasetGrid.cpp.o ModelDatasetGrid.h ModelDatasetGenerator.h PhzModel.h  (1907 ms)
  GalacticCorrectionFactorSingleGridCreator.cpp.o ModelDatasetGrid.h ModelDatasetGenerator.h PhzModel.h  (1871 ms)
  ParameterSpaceConfig.cpp.o ParameterSpaceConfig.h PhzModel.h  (1847 ms)
  ModelDatasetGenerator.cpp.o ModelDatasetGenerator.h PhzModel.h  (1823 ms)
  PhzModel.cpp.o PhzModel.h  (1789 ms)
  GenericGridPrior_test.cpp.o GenericGridPrior.h DoubleGrid.h PhzModel.h  (1761 ms)
  ...

157469 ms: /home/aalvarez/Work/Projects/PhosphorosCore/PhosphorosCore/PhzDataModel/PhzDataModel/PhzModel.h (included 135 times, avg 1166 ms), included via:
  GenericGridPrior_test.cpp.o GenericGridPrior.h DoubleGrid.h  (2239 ms)
  PhzModel.cpp.o  (2231 ms)
  SingleGridPhzFunctor.cpp.o SingleGridPhzFunctor.h DoubleGrid.h  (2177 ms)
  SumMarginalizationFunctor_test.cpp.o SumMarginalizationFunctor.h DoubleGrid.h  (2159 ms)
  LikelihoodGridFunctor.cpp.o LikelihoodGridFunctor.h DoubleGrid.h  (2112 ms)
  MaxMarginalizationFunctor_test.cpp.o MaxMarginalizationFunctor.h DoubleGrid.h  (2110 ms)
  ...

145619 ms: /usr/include/boost/program_options.hpp (included 134 times, avg 1086 ms), included via:
  MarginalizationConfig.cpp.o MarginalizationConfig.h Configuration.h  (1816 ms)
  PdfOutputFlagsConfig.cpp.o PdfOutputFlagsConfig.h Configuration.h  (1815 ms)
  ModelGridOutputConfig.cpp.o ModelGridOutputConfig.h Configuration.h  (1607 ms)
  SedProviderConfig_test.cpp.o ConfigManager_fixture.h ConfigManager.h  (1587 ms)
  AxisFunctionPriorConfig_test.cpp.o ConfigManager_fixture.h ConfigManager.h  (1563 ms)
  MultithreadConfig.cpp.o MultithreadConfig.h Configuration.h  (1559 ms)
```

Nice! We can see that the compiler spends 173 seconds (!) just parsing
`PhotometryGrid.h`, 161s `GridContainer/serialize.h`, etc.

The first insight is: why is a serialization header included 135 times?
There should not be many units concerned with writing or reading
the grid. And, indeed, there aren't. `PhzModel.h` is overreaching.

Easy fix, just split the serialization code contained in `PhzModel.h` into
another header, and include it only on the sources that care about IO.

What about `program_options.hpp`? Well, `Configuration.h` includes it
since it handles argument parsing, *but* in reality, it only needs
`boost::program_options::options_description` and `boost::program_options::variable_value`.
Let's remove the inclusion of `program_options.hpp` and include only
`boost/program_options/options_description.hpp` and `boost/program_options/variables_map.hpp`.

We can see the idea, include the minimum possible. With this, I cut the
build time by 10%.

Still, `Configuration.h` from Alexandria was particularly heavy, due to the
inclusion of `boost/program_options/options_description.hpp`. I had never tried
precompiled headers, but I decided to try it since many files in PhosphorosCore
include `Configuration.h`.

```cmake
if (${CMAKE_VERSION} VERSION_GREATER "3.16.0" OR ${CMAKE_VERSION} VERSION_EQUAL "3.16.0")
    target_precompile_headers(PhzConfiguration PRIVATE
            <Configuration/Configuration.h>
            <Configuration/ConfigManager.h>
            <GridContainer/serialize.h>)
endif ()
```

This cut down the build time further! The total savings now is at 25%!

That was worth it 😄
