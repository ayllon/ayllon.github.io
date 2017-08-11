---
title: The dangers of the singletons (or lots of them)
---

Quite some time ago I wrote on the FTS blog an entry about what I had
to do to [extract test coverage](http://fts3-service.web.cern.ch/content/extracting-coverage-information-fts3)
from the service.

One thing I didn't mention, though, it's how I was bitten by having a bunch
of singletons flying around. Combined with different possible exiting paths
(i.e. the expected one, signals, etc.) that made it harder.

## The details

I did what I mentioned on that blog entry, but I had a small problem: when the
program exited, no coverage information *at all* would be generated. Not
even for `main`, which is obviously wrong.

The problem was that the coverage data is written to disk when the program exits
normally. When you call `exit`, or `return` from main, there is a cleaning up process,
and a set of handlers installed via [`atexit`](http://man7.org/linux/man-pages/man3/atexit.3.html)
are called. One of these handlers is installed by gcov to dump the coverage information.

FTS, being a service, runs in the background, and installs a signal handler for,
between others, `SIGTERM`, so the service can be shutdown. This handler
would stop the subservices, clean a bit up, and call [`_exit`](http://man7.org/linux/man-pages/man2/exit.2.html).

Let's check the documentation about `_exit`

>  The function \_exit() is like exit(3), but does not call any functions
> registered with atexit(3) or on_exit(3)

Ooops. That explains it. Case solved! Just call `exit`!

## Singletons go kicking
It wasn't solved. Once I changed all `_exit` calls with `exit`, I would get
crashes every time I tried to shutdown the service, and no coverage information
would be generated either.

The reason behind was that a bunch of singletons were referencing each other.
Let's imagine we have a Logger singleton, and some Factory singleton.

The Factory singleton calls the logger when created or destroyed. This may
be a silly idea, but bear with me. It could as well do something with a database
backend that then calls the logger, or whatever.

```cpp
#include <iostream>
#include <memory>
#include <unistd.h>
#include <boost/core/noncopyable.hpp>

template <class C>
class Singleton: public boost::noncopyable {
protected:
  Singleton() {}
public:
  static C& getInstance() {
    static std::unique_ptr<C> instance;
    if (!instance) {
      instance.reset(new C);
    }
    return *instance.get();
  }
};

class Logger: public Singleton<Logger> {
private:
  // Just to get my point across
  std::ostream *out;

public:
  Logger() {
    out = &std::cout;
  }

  ~Logger() {
    *out << "Logger destroyed" << std::endl;
  }

  void print(std::string const &str) {
    *out << str << std::endl;
  }
};

class Factory: public Singleton<Factory> {
public:
  Factory() {
    Logger::getInstance().print("Factory created");
  }

  ~Factory() {
    Logger::getInstance().print("Factory destoyed");
  }

  void doSomething(void) {
    Logger::getInstance().print("Just did something");
  }
};

int main(int, char**)
{
  Factory::getInstance().doSomething();
  return 0;
}
```

That will crash (at least when compiled with -O1 with g++ 6.3) just after main
returns. It would too if `exit` is called.

```
$ ./a.out
Factory created
Just did something
Logger destroyed
Segmentation fault (core dumped)
```

Some familiar with C++ and its guarantees will see immediately the reason.
Those that think this code makes no sense, again, this is an artificial
example to prove that, indeed, it is nonsense.

So the problem here is that C++ guarantees that the destruction order is
*the reverse* of the construction order, which makes sense.

The call

```cpp
Factory::getInstance()
```

creates the Factory instance. The constructor of the Factory then calls

```cpp
Logger::getInstance()
```

 which creates the logger. Therefore, the destruction order is going to be
 the reverse: first the Logger, then the Factory. But the Factory destructor
 uses the Logger! And that's when the crash happens.

That's probably why the code was sprinkled with `_exit` calls. In that case,
no destructor is called, and all is good, but not particularly elegant.

The crash above can be "solved" if you just try to log something before creating
the factory:

```cpp
int main(int, char**)
{
  Logger::getInstance().print("Starting");
  Factory::getInstance().doSomething();
  return 0;
}
```

Of course, that's not a good solution either. What I did is to get rid of as
many singletons as I could, turn them into attributes, and pass them down.

Eventually I managed to get rid of all these unobvious dependencies, and `exit`
now, well, *exits* cleanly, and the coverage data is written to disk.

## Conclusion
Avoid singletons, and please, avoid singletons that depend on other singletons.
