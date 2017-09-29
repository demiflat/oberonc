
# Oberon-07 compiler

`oberonc` is a single pass, self-hosting compiler for the
[Oberon-07](https://en.wikipedia.org/wiki/Oberon_(programming_language))
programming language. It targets the Java Virtual Machine (version >=1.5).

This project was started to showcase Niklaus Wirth's approach to writing
compilers (see
["Compiler Construction -
The Art of Niklaus Wirth"](http://www.oberon2005.oberoncore.ru/paper/art3.pdf)
by Hanspeter Mössenböck for more details).

`oberonc` is inspired by Niklaus Wirth's compiler for a RISC processor available
[here](http://www.inf.ethz.ch/personal/wirth/).

The compiler is compact and does not depend on any third party libraries. It
produces Java bytecode in one pass while parsing the source file. Although
generating code for a stack machine is straightforward, this task is exacerbated
by a complex class file format and the fact that the JVM was designed with the
Java language in mind. In fact the JVM lacks many of the primitives required to
support Oberon's features, specifically:

* value types
* pass by reference evaluation strategy
* procedure variables (pointer to functions) and relative structural
  compatibility of types

Implementing those features with workarounds increased significantly the size
of the compiler, totaling roughly 6000 lines of Oberon.

The source code is written following as much as possible Niklaus Wirth's
coding style. `oberonc` compile itself in less than 300 ms on an old
Intel i5 @ 2.80GHz (~ 100 ms with a hot VM).

## How to build

You can build the compiler on Linux or Windows, you need a JDK >= 1.8
installed, with java and javac in the environment path.

First of all, you need to set the OBERON_BIN environmental variable to the `bin`
folder of the repository, for
example on Linux `export OBERON_BIN=~/projects/oberonc/bin` or
`set OBERON_BIN=C:\oberonc\bin` on Windows. Because you need an Oberon compiler
to compile the sources in `src`, I have added to the repository the binaries of
the compiler to perform the bootstrapping.

By typing `make build` on the shell, the compiler will compile itself and
overwrite the files in the `bin` folder.

## How to run the tests

One typical test is to make sure that, by compiling the compiler, we get the
same (bit by bit) class files originally included in the `bin` folder.
To run this test simply type `make bootstrapTest` (available only on Linux).
This will compile the sources into the `bootstrapOut` folder and compare these
resulting class files with the ones in `bin`. If something goes wrong
`sha1sums` will complain.

To run the tests included in the `tests` folder, type `make test`. The output
should look like this:

    ...
    TOTAL: 89
    SUCCESSFUL: 89
    FAILED: 0

## Using the compiler

To use the compiler, you need to have the OBERON_BIN variable set to the `bin`
folder of the repository. The command line syntax of `oberonc` is simple.
Let's compile examples/Hello.Mod:

    MODULE Hello;
      IMPORT Out; (* Import Out to print on the console *)
    BEGIN
      Out.String("Hello 世界");
      Out.Ln (* print a new line *)
    END Hello.

Assuming you are at the root of the repository, the following command will
compile the Hello.Mod example and place the generated classes in the current
folder:

    Linux
    java -cp $OBERON_BIN oberonc . examples/Hello.Mod

    Windows
    java -cp %OBERON_BIN% oberonc . examples/Hello.Mod

The first argument of oberonc is `.`, this is the existing folder where the
generated class will be written, the next arguments specify module files to
be compiled.

This will generate Hello.class and Hello.smb. The second file is a symbol file,
it is used only during compilation and enables `oberonc` to perform separate
compilation of modules that import Hello. In this simple case Hello.Mod
does not export anything, but the other modules in the `examples` folder do.

To run Hello.class, you need the OberonRuntime.class and Out.class. These are
present in the `bin` folder so they are already in the class path, we just need
to include the current folder as well to locate Hello.class:

    Linux
    java -cp $OBERON_BIN:. Hello

    Windows
    java -cp %OBERON_BIN%;. Hello

If you want to compile and run automatically a simple example called `fern`,
type `make runFern`. It should open a window like this one:

![Fern](examples/fern/fern.png)

Lastly, `make clean` will delete the output folders generated by `test`,
`runFern` and `bootstrapTest`.

## License

The compiler is distributed under the MIT license found in the LICENSE.txt file.
