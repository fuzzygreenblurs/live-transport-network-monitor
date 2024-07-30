## The Problem

- everyday, the London Tube handles ~5M passenger journeys
- it has 11 lines and 270 stations
- passenger load varies throughout the day and depends on several factors

  - ex: time of the day, the area, the Tube line

- The London Transport Company (LTC) wants to manage passenger load on the busiest lines to avoid overcrowding

  - LTC can measure the network load in real-time

    - each journey starts and ends by tapping an electronic card at the station gates
    - therefore, the origin and destination of every trip is recorded

  - LTC can influence the itinerary choice of many passengers
    - LTC provides live itinerary recommendations to map services like Google Maps, Apple Maps, Citymapper, etc.
    - when a user searches how to get from A to B, the Tube itinerary is suggested by the LTC systems
    - we will assume that 15% of all trips are driven by these map services (~750K passenger journeys)

- target: LTC wants to implement an itinerary recommendation engine

  - it will generatee Tube trips based on real-time network load
  - these itineraries are called quiet routes
  - in general they should be:
    - noticeably less busy
    - not much slower than the fastest route available
      - a quiet route makes sense only if the fastest route is very busy
      - otherwise people should just go for the fastest route

#### Order of Implementation

- at first, LTC will only offer the quiet route option to a small subset of requests for recommendations
  — ~20% of all user itinerary recommendations

  - thus, our implementation does not have to sustain an incredible amount of user requests
  - note: it still needs to handle all network events from ~5 million daily journeys that take place on the Tube
    - this means processing the enter/exit scanning events for each journey (recommended or not)

## Project Structure and Delivery

1. Research:

- formalize the problem into specific requirements and constraints
- research the technologies we'll use
- design a high-level architecture of the system to identify the main building blocks we will implement

2. Playground:

- get familiar with the components and technologies we will use throughout the Project
- start with sparse code snippets as a good starting point for the rest of the Project

3. Functionality:

- take the code from the Playground phase and give it some structure
- implement all the functionality needed and make sure it works as expected by implementing many test cases

4. Optimization:

- once we have a working system, start measuring its critical paths and performance
- use measurements to decide if and how to optimize parts of the code

5. Productization:

- polish code, tooling, and build system
- this ensures it can go into production and it can be maintained with a high level of quality

Notes:

- at each stage, we give you all the background you need
- this includes C++ concepts, syntax, libraries and programming techniques
  — but you will still have to write the code

- we will ask you to adhere to some conventions
  - ex: we ask you to use specific APIs from time to time

## Conventions

- project notes contain several different ingredients:
  - C++ theory
  - code snippets
  - API recommendations
  - etc.
- we use visual cues to show you what kind of information you are reading

  - regular paragraph: project notes (focused guide discussing topics/technologies to implement the project)

    - ex: instructions to follow (create new files, copy-paste code, etc.)
    - it will come with a REQUIRED badge

  - there will be sections marked with "your turn":

    - we give you some constraints when asking you to implement code
    - these constraints are designed to make sure that we can make progress together

      - this still gives you as much freedom as possible to develop your own implementation

    - sometimes, we also include hints to get you started, especially if we ask you to work on a new topic
    - every time that we ask you to implement a Project step, we follow up immediately with our own answer

  - in depth:
    - we try to keep our Project notes as on-topic as possible
    - sometimes a topic is simply too vast or nuanced to be presented in a couple of paragraphs
    - when we think you can benefit from some in-depth analysis, we include orange "in-depth" sections

## Tooling

#### Intro

- our recommended C++ development environment uses several tools:

  - compiler/linker (it depends on your operating system: GCC, MSVC, or Apple Clang)
  - build system (ninja)
  - build generator (CMake)
  - dependency manager (conan). conan in turn requires Python 3
  - version control software (git)

- if you have built C++ code before you may already have many of these tools
  - read through the instructions regardless!
  - make sure that you have the correct version of each tool and that it works as expected

#### Compiler

- C++ is a compiled language

  - you cannot run C++ code directly
  - you first need to compile it into a format that can be executed

- compilation is a multi-stage process

  - it takes the source files and turns them into an output artifact (ex: an executable or a shared library)

- the steps of the compilation process go from preprocessing to linking and are performed by different tools
- all these tools are usually installed together as part of a compiler package

- you can build C++ code virtually on any operating system

  - your specific setup may affect the version of the C++ standard that you can build with

- we use features from the C++11, C++14 and C++17 standards

  - use a compiler version released — as a rule of thumb — on or after January 2020 (>=Clang 12, >=gcc 10)
  - mainly, make sure your compiler fully supports c++17
  - [Compilers supporting C++17](https://en.cppreference.com/w/cpp/compiler_support#cpp17)
  - [C++17 Support in GCC](https://gcc.gnu.org/projects/cxx-status.html#cxx17)

- to check compiler version do:

```sh
clang++ --version
g++ --version # Equivalent

---- yields ----
Apple clang version 13.1.6 (clang-1316.0.21.2.5)
Target: arm64-apple-darwin21.3.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

- the g++ command also links to the Apple Clang compiler

  - Clang is command-compatible to GCC — it can be used as a drop-in replacement for GCC

- to test that the compiler is C++17 compatible, try to compile:

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::pair<std::string, double> pair {"Hello, World", 42};

    // Structured bindings are a C++17 feature.
    const auto& [str, num] = pair;
    std::cout << str << ": " << num << std::endl;

    return 0;
}
```

To compile and run:

```sh
clang++ --std=c++17 -o test.out test.cpp
./test.out

--- yields ---
Hello, World: 42
```

#### Build System

- the compilation step is a very complex one
- you can configure the compiler to build your source code with many different:

  - configuration options
  - optional flags
  - custom preprocessor definitions
  - etc.

- to keep track of the rules and options to build our Project source files, we use a build system
- our build system of choice is ninja, which is a C++ industry standard
- ninja uses a set of scripts to execute compiler commands and generate artifacts as a result

  - when a source file changes, ninja knows:
    - which files need to be rebuilt
    - which artifacts need to be regenerated

- the set of rules that defines how to build a group of source files is called a build target

  - writing rules for the build system manually would be very cumbersome, complex
  - it is also hard to make portable

- instead of writing build system rules manually, we use a tool for defining targets — a build system generator
  - The build generator of our choice is CMake, another industry standard

#TO_DO: what is ninja? what is CMake? how do they differ in purpose? how do we use them practically?

#### Package Manager

- very often, a Project includes C++ code from third parties

  - because of the way C++ code is built, it is not always easy to integrate third-party code with our code

- We use a package manager to simplify this task, and will also use it to package and deploy our software so that other people can integrate it seamlessly in their projects.

- our package manager of choice is conan. conan requires Python 3 to be installed on your machine.

  - We use conan quite a lot in this Project:
    - to include third party software
    - to package our software for distribution

- conan is very popular right now, but it is still not a universally accepted industry standard

  - we integrate it in a way that makes it optional
  - you can always revert to other packaging systems or use pure CMake commands to import third-party code

```sh

(venv) ➜ pip3 install conan
(venv) ➜ conan --version

(venv) ➜ conan profile detect
detect_api: Found apple-clang 13.1
detect_api: apple-clang>=13, using the major as version

Detected profile:
[settings]
arch=armv8
build_type=Release
compiler=apple-clang
compiler.cppstd=gnu17
compiler.libcxx=libc++
compiler.version=13
os=Macos

WARN: This profile is a guess of your environment, please check it.
WARN: Defaulted to cppstd='gnu17' for apple-clang.
WARN: The output of this command is not guaranteed to be stable and can change in future Conan versions.
WARN: Use your own profile files for stability.
Saving detected profile to /Users/akhilsankar/.conan2/profiles/default

```
