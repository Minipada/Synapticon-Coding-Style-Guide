
|image/image0.gif|



====================
CODING GUIDE C & C++
====================

Version 1.0

SYNAPTICON

--------------

Document Change Log
===================

+-------------------------+-------------------------+-------------------------+
| Description             | Name                    | Date                    |
+-------------------------+-------------------------+-------------------------+
| Start of document       | Levent Yildirim         | 21/06/2015              |
+-------------------------+-------------------------+-------------------------+
| Major Adaption          | David Bensoussan        | 2015-08-06              |
+-------------------------+-------------------------+-------------------------+

--------------

Table Of Content

.. contents::


--------------


Content
=======

This document is the official coding guideline of Synapticon GmbH. All
developments, which are done by Synapticon or are done by third party
for Synapticon has to follow this coding guideline. This coding
guideline covers the programming languages of C and C++.

Parts of the guideline which can´t be used in C, but in C++ are only
binding to follow this guideline in C++ and vice versa.

The content of this coding guideline based on the coding guideline of
Google C++ Style Guide. The content is mainly copied from Google C++
Style Guide. There are only few changes and a lot of content which is
not included in this guideline.

--------------

Header files
============

In general, every ``.cpp`` file should have an associated ``.h`` file. There
are some common exceptions, such as unittests and small ``.cpp`` files
containing just a ``main()`` function.

Correct use of header files can make a huge difference to the
readability, size and performance of your code.

The following rules will guide you through the various pitfalls of using
header files.

Self-contained Headers
----------------------

Header files should be self-contained and end in ``.h``. Files that are
meant for textual inclusion, but are not headers, should end in ``.inc``.
Separate-inl.h headers are disallowed.

All header files should be self-contained. In other words, users and
refactoring tools should not have to adhere to special conditions in
order to include the header. Specifically, a header should have
header-guards, should include all other headers it needs, and should not
require any particular symbols to be defined.

There are rare cases where a file is not meant to be self-contained, but
instead is meant to be textually included at a specific point in the
code. Examples are files that need to be included multiple times or
platform-specific extensions that essentially are part of other headers.
Such files should use the file extension ``.inc``.

If a template or inline function is declared in a ``.h`` file, define it in
that same file. The definitions of these constructs must be included
into every ``.cpp`` file that uses them, or the program may fail to link in
some build configurations. Do not move these definitions to
separate-inl.h files.

As an exception, a function template that is explicitly instantiated for
all relevant sets of template arguments, or that is a private member of
a class, may be defined in the only ``.cpp`` file that instantiates the
template.

No code in inc files!

The #define Guard
-----------------

All header files should have #define guards to prevent multiple
inclusion. The format of the symbol name should be
<PROJECT>\_<PATH>\_<FILE>\_H\_. Important is that names are different.

To guarantee uniqueness, they should be based on the full path in a
project's source tree. For example, the file foo/src/bar/baz.h in
project foo should have the following guard:

:: 

        #ifndef FOO\_BAR\_BAZ\_H\_

        #define FOO\_BAR\_BAZ\_H\_

        …

        #endif  // FOO\_BAR\_BAZ\_H\_

XMOS uses ``#pragma once``

--------------

Inline Functions
----------------

Define functions inline only when they are small

Function Parameter Ordering
---------------------------

When defining a function, parameter order is: inputs, then outputs.

Parameters to C/C++ functions are either input to the function, output
from the function, or both. Input parameters are usually values or const
references, while output and input/output parameters will be
non-const pointers. When ordering function parameters, put all
input-only parameters before any output parameters. In particular, do
not add new parameters to the end of the function just because they are
new; place new input-only parameters before the output parameters.

This is not a hard-and-fast rule. Parameters that are both input and
output (often classes/ structs) muddy the waters, and, as always,
consistency with related functions may require you to bend the rule.

XMOS: Need more rules for interfaces, ports and channels. Interfaces and
channels come first. Ports come second. All of their arguments follow
like described in the C/C++ guideline.

Names and Order of Includes
---------------------------

Use standard order for readability and to avoid hidden dependencies:
Related header, C library, C++ library, other libraries' .h, your
project's .h.

All of a project's header files should be listed as descendants of the
project's source directory without use of UNIX directory shortcuts. (the
current directory) or .. (the parent directory). For example,
synapticon-awesome-project/src/base/logging.h should be included as:

:: 
        #include "base/logging.h"

In dir/foo.cpp or dir/foo\_test.cpp, whose main purpose is to implement
or test the stuff in dir2/foo2.h, order your includes as follows:

#. dir2/foo2.h.
#. Other libraries'.h files.
#. Your project's .h files.
#. C system files.
#. C++ system files.

For example, the includes in
``synapticon-awesome-project/src/foo/internal/fooserver.cpp``
 might look like this::

        #include "foo/server/fooserver.h"

        #include "base/basictypes.h"

        #include "base/commandlineflags.h"

        #include "foo/server/bar.h"

        #include <sys/types.h>

        #include <unistd.h>

        #include <hash\_map>

        #include <vector>

Exception:

Sometimes, system-specific code needs conditional includes. Such code
can put conditional includes after other includes. Of course, keep your
system-specific code small and localized.

Example::

        #include "foo/public/fooserver.h"

        #include "base/port.h"  // For LANG\_CXX11.

        #ifdef LANG\_CXX11

        #include <initializer\_list>

        #endif  // LANG\_CXX11

--------------

Scoping
=======

Namespaces
----------

Unnamed namespaces in .cpp files are encouraged. With named namespaces,
choose the name based on the project, and possibly its path. Do not use
a using-directive. Do not use inline namespaces.

Unnamed Namespaces
~~~~~~~~~~~~~~~~~~

-  Unnamed namespaces are allowed and even encouraged in .cpp files, to
   avoid link time naming conflicts::
        namespace {                           // This is in a .cpp file.
        
        // The content of a namespace is not indented.
        //
        // This function is guaranteed not to generate a colliding symbol
        // with other symbols at link time, and is only visible to
        // callers in this .cpp file.
        bool UpdateInternals(Frobber\* f, int newval) {
          ...
        }
        
        }  // namespace

However, file-scope declarations that are associated with a particular
class may be declared in that class as types, static data members or
static member functions rather than as members of an unnamed namespace.

-  Do not use unnamed namespaces in .h files.

Named Namespaces
~~~~~~~~~~~~~~~~

Named namespaces should be used as follows:

-  Namespaces wrap the entire source file after includes,gflags
   definitions/declarations, and forward declarations of classes from
   other namespaces::

        // In the .h file
        namespace mynamespace {

        // All declarations are within the namespace scope.
        // Notice the lack of indentation.
        class MyClass {
         public:
          ...
          void Foo();
        };

        }  // namespace mynamespace
        // In the .cpp file
        namespace mynamespace {

        // Definition of functions is within scope of the namespace.
        void MyClass::Foo() {
          ...
        }

        }  // namespace mynamespace

The typical .cpp file might have more complex detail, including the need
to reference classes in other namespaces.

::
        #include "a.h"

        DEFINE\_bool(someflag, false, "dummy flag");

        class C;  // Forward declaration of class C in the global namespace.

        namespace a { class A; }  // Forward declaration of a::A.

        namespace b {

        ...code for b...         // Code goes against the left margin.

        }  // namespace b

-  You may use a using-declaration anywhere in a .cpp file, and in
   functions, methods or classes in .h files.

::

        // OK in .cpp files.

        // Must be in a function, method or class in .h files.

        using ::foo::bar;

-  Do not use inline namespaces.

--------------

Nonmember, Static Member, and Global Functions
----------------------------------------------

Prefer nonmember functions within a namespace or static member functions
to global functions; use completely global functions rarely.

Local variables
---------------

C++ allows you to declare variables anywhere in a function. We encourage
you to declare them in as local a scope as possible, and as close to the
first use as possible. This makes it easier for the reader to find the
declaration and see what type the variable is and what it was
initialized to. In particular, initialization should be used instead of
declaration and assignment, e.g.:

::

        int i;

        i = f();      // Bad -- initialization separate from declaration.

        int j = g();  // Good -- declaration has initialization.

        vector<int> v;

        v.push\_back(1);  // Prefer initializing using brace initialization.

        v.push\_back(2);

        vector<int> v = {1, 2};  // Good -- v starts initialized.

Static and Global Variables
---------------------------

Static or global variables of class type are forbidden: they cause
hard-to-find bugs due to indeterminate order of construction and
destruction. However, such variables are allowed if they are constexpr:
they have no dynamic initialization or destruction.

Objects with static storage duration, including global variables, static
variables, static class member variables, and function static variables,
must be Plain Old Data (POD): only ints, chars, floats, or pointers, or
arrays/ structs of POD.

--------------

Classes
=======

Classes are the fundamental unit of code in C++. Naturally, we use them
extensively. This section lists the main dos and don'ts you should
follow when writing a class.

Doing Work in Constructors
--------------------------

Avoid doing complex initialization in constructors (in particular,
initialization that can fail or that requires virtual method calls).

Constructors should never call virtual functions or attempt to raise
non-fatal failures. If your object requires non-trivial initialization,
consider using a factory function or Init() method.

Initialization
--------------

If your class defines member variables, you must provide an in-class
initializer for every member variable or write a constructor (which can
be a default constructor). If you do not declare any constructors
yourself then the compiler will generate a default constructor for you,
which may leave some fields uninitialized or initialized to
inappropriate values.

Delegating and Inheriting Constructors
--------------------------------------

Use delegating and inheriting constructors when they reduce code
duplication.

::

        X::X(const string& name)

        : name\_(name)

        , var\_name(var)

        {

          ...

        }

        X::X() : X("") { }

        class Base {

         public:

          Base();

          Base(int n);

          Base(const string& s);

          ...

        };

        class Derived : public Base {

         public:

          using Base::Base;  // Base's constructors are redeclared here.

        };

Structs vs. Classes
-------------------

Use a struct only for passive objects that carry data; everything else
is a class.

Structs should be used for passive objects that carry data, and may have
associated constants, but lack any functionality other than
access/setting the data members. The accessing/setting of fields is done
by directly accessing the fields rather than through method invocations.
Methods should not provide behavior but should only be used to set up
the data members, e.g., constructor, destructor, Initialize(), Reset(),
Validate().

If more functionality is required, a class is more appropriate. If in
doubt, make it a class.

Interfaces
----------

Classes that satisfy certain conditions are allowed, but not required,
to end with an Interface suffix.

Operator Overloading
--------------------

Do not overload operators except in rare, special circumstances. Do not
create user-defined literals.

Write Short Functions
---------------------

Prefer small and focused functions.

We recognize that long functions are sometimes appropriate, so no hard
limit is placed on functions length. If a function exceeds about 40
lines, think about whether it can be broken up without harming the
structure of the program.

--------------

Naming
======

The most important consistency rules are those that govern naming. The
style of a name immediately informs us what sort of thing the named
entity is: a type, a variable, a function, a constant, a macro, etc.,
without requiring us to search for the declaration of that entity. The
pattern-matching engine in our brains relies a great deal on these
naming rules.

General Naming Rules
--------------------

Function names, variable names, and filenames should be descriptive;
eschew abbreviation.

Give as descriptive a name as possible, within reason. Do not worry
about saving horizontal space as it is far more important to make your
code immediately understandable by a new reader. Do not use
abbreviations that are ambiguous or unfamiliar to readers outside your
project, and do not abbreviate by deleting letters within a word.

::

        int price\_count\_reader;    // No abbreviation.

        int num\_errors;            // "num" is a widespread convention.

        int num\_dns\_connections;   // Most people know what "DNS" stands for.

        int n;                     // Meaningless.

        int nerr;                  // Ambiguous abbreviation.

        int n\_comp\_conns;          // Ambiguous abbreviation.

        int wgc\_connections;       // Only your group knows what this stands
        for.

        int pc\_reader;             // Lots of things can be abbreviated "pc".

        int cstmr\_id;              // Deletes internal letters.

File Names
----------

Filenames should be all lowercase and can include underscores (\_) or
dashes (-). Follow the convention that your project uses. If there is no
consistent local pattern to follow, prefer "\_".

Examples of acceptable file names:

-  my\_useful\_class.cpp
-  my-useful-class.cpp
-  myusefulclass.cpp
-  myusefulclass\_test.cpp // \_unittest and \_regtest are deprecated.

C++ files should end in.cpp and header files should end in.h. Files that
rely on being textually included at specific points should end in.inc

Do not use filenames that already exist in /usr/include, such as db.h.

In general, make your filenames very specific. For example, use
http\_server\_logs.h rather than logs.h. A very common case is to have a
pair of files called, e.g., foo\_bar.h and foo\_bar.cpp, defining a
class called FooBar.

Type Names
----------

Type names start with a capital letter and have a capital letter for
each new word, with no underscores: MyExcitingClass, MyExcitingEnum.

The names of all types — classes, structs, typedefs, and enums — have
the same naming convention. Type names should start with a capital
letter and have a capital letter for each new word. No underscores. For
example:

::

        // classes and structs

        class UrlTable { ...

        class UrlTableTester { ...

        struct UrlTableProperties { ...

        // typedefs

        typedef hash\_map<UrlTableProperties \*, string> PropertiesMap;

        // enums

        enum UrlTableErrors { ...

Variable Names
--------------

The names of variables and data members are all lowercase, with
underscores between words. Data members of classes (but not structs)
additionally have trailing underscores. For instance:
a\_local\_variable, a\_struct\_data\_member, a\_class\_data\_member\_.

For example::

        string table\_name;  // OK - uses underscore.

        string tablename;   // OK - all lowercase.

        string tableName;   // Bad - mixed case.

Class Data Members
------------------

Data members of classes, both static and non-static, are named like
ordinary nonmember variables, but with a trailing underscore.

::

        class TableInfo {

          ...

         private:

          string table\_name\_;  // OK - underscore at end.

          string tablename\_;   // OK.

          static Pool<TableInfo>\* pool\_;  // OK.

        };

Struct Data Members
-------------------

Data members of structs, both static and non-static, are named like
ordinary nonmember variables. They do not have the trailing underscores
that data members in classes have.

::

        struct UrlTableProperties {

          string name;

          int num\_entries;

          static Pool<UrlTableProperties>\* pool;

        };

Global Variables
----------------

There are no special requirements for global variables, which should be
rare in any case, but if you use one, consider prefixing it with g\_or
some other marker to easily distinguish it from local variables.

Constant Names
--------------

Constants should be completely uppercase, for constants defined globally
or within a class.

Separate word by \_ underscore:

::

        const int DAYS\_IN\_WEEK = 7;

Regular Functions
~~~~~~~~~~~~~~~~~

Functions should start with a capital letter and have a capital letter
for each new word. No underscores.

If your function crashes upon an error, you should append OrDie to the
function name. This only applies to functions which could be used by
production code and to errors that are reasonably likely to occur during
normal operation.

C++

::

        addTableEntry()
        deleteUrl()
        openFileOrDie()

C

::

        add\_table\_entry()
        delete\_url()
        open\_file\_or\_die()

Accessors and Mutators
~~~~~~~~~~~~~~~~~~~~~~

Accessors and mutators (get and set functions) should match the name of
the variable they are getting and setting. This shows an excerpt of a
class whose instance variable is num\_entries\_.

::

        class MyClass {

         public:

          ...

          int getNumEntries() const { return num\_entries\_; }

          void setNumEntries(int num\_entries) { num\_entries\_ = num\_entries;
        }

          bool isActive()

         private:

          int num\_entries\_;

        };

You may also use lowercase letters for other very short inlined
functions. For example if a function were so cheap you would not cache
the value if you were calling it in a loop, then lowercase naming would
be acceptable.

Macro Names
-----------

In general macros should not be used. However, if they are absolutely
needed, then they should be named with all capitals and underscores.

::

        #define ROUND(x) ...

        #define PI\_ROUNDED 3.0

--------------

Comments
========

When writing your comments, write for your audience: the next
contributor who will need to understand your code. Be generous — the
next one may be you!

Comment as less as possible and as much as needed. By using good naming
it should be possible to reduce comments to a minimum.

Comment Style
-------------

Use either the // or /\* \*/ syntax, as long as you are consistent.

You can use either the // or the /\* \*/ syntax; however, // is
much more common. Be consistent with how you comment and what style you
use where.

Explanatory Comments
--------------------

Tricky or complicated code blocks should have comments before them.
Example:

::

        // Divide result by two, taking into account that x

        // contains the carry from the add.

        for (int i = 0; i < result->size(); i++) {

          x = (x << 8) + (\*result)[i];

          (\*result)[i] = x >> 1;

          x &= 1;

        }

nullptr/NULL, true/false, 1, 2, 3...
------------------------------------

When you pass in a null pointer, boolean, or literal integer values to
functions, you should consider adding a comment about what they are, or
make your code self-documenting by using constants. For example,
compare:

::

        bool success = CalculateSomething(interesting\_value,

                                          10,

                                          false,

                                          NULL);  // What are these arguments??

versus::

        bool success = CalculateSomething(interesting\_value,

                                          10,     // Default base value.

                                          false,  // Not the first time we're calling this.

                                          NULL);  // No callback.

Or alternatively, constants or self-describing variables::

        const int kDefaultBaseValue = 10;
        const bool kFirstTimeCalling = false;
        Callback \*null\_callback = NULL;

        bool success = CalculateSomething(interesting\_value,

                                          kDefaultBaseValue,

                                          kFirstTimeCalling,

                                          null\_callback);

Don'ts
------

Note that you should never describe the code itself. Assume that the
person reading the code knows C++ better than you do, even though he or
she does not know what you are trying to do::

        // Now go through the b array and make sure that if i occurs,

        // the next element is i+1.

        ...        // Geez.  What a useless comment.

--------------

Formatting
==========

Coding style and formatting are pretty arbitrary, but a project is much
easier to follow if everyone uses the same style. Individuals may not
agree with every aspect of the formatting rules, and some of the rules
may take some getting used to, but it is important that all project
contributors follow the style rules so that they can all read and
understand everyone's code easily.

Line Length
-----------

Each line of text in your code should be at most 80 characters long,
exceptions are allowed.

Spaces vs. Tabs
---------------

Use only spaces, and indent 2 spaces at a time.

We use spaces for indentation. Do not use tabs in your code. You should
set your editor to emit spaces when you hit the tab key.

Function Declarations and Definitions
-------------------------------------

Return type on the same line as function name, parameters on the same
line if they fit. Wrap parameter lists which do not fit on a single line
as you would wrap arguments in a function call.

Functions look like this::

        ReturnType ClassName::FunctionName(Type par\_name1, Type par\_name2)
        {

          DoSomething();

          ...

        }

If you have too much text to fit on one line::

        ReturnType ClassName::ReallyLongFunctionName(Type par\_name1,

                                                     Type par\_name2,

                                                     Type par\_name3)

        {

          DoSomething();

          ...

        }

If some parameters are unused, comment out the variable name in the
function definition::

        // Always have named parameters in interfaces.

        class Shape {

         public:

          virtual void Rotate(double radians) = 0;

        };

        // Always have named parameters in the declaration.

        class Circle : public Shape {

         public:

          virtual void Rotate(double radians);

        };

        // Comment out unused named parameters in definitions.

        void Circle::Rotate(double /\*radians\*/) {}

        // Bad - if someone wants to implement later, it's not clear what the
        // variable means.

        void Circle::Rotate(double) {}

Function Calls
--------------

Either write the call all on a single line, wrap the arguments at the
parenthesis, or start the arguments on a new line indented by four
spaces and continue at that 4 space indent. In the absence of other
considerations, use the minimum number of lines, including placing
multiple arguments on each line where appropriate.

Function calls have the following format::

        bool retval = DoSomething(argument1, argument2, argument3);

If the arguments do not all fit on one line, they should be broken up
onto multiple lines, with each subsequent line aligned with the first
argument. Do not add spaces after the open paren or before the close
paren::

        bool retval = DoSomething(averyveryveryverylongargument1,

                                  argument2,

                                  argument3);

Arguments may optionally all be placed on subsequent lines ``if (...) {``

::

          ...

          if (...) {

            DoSomething(argument1,

                        argument2,

                        argument3,

                        argument4);

          }

Put multiple arguments on a single line to reduce the number of lines
necessary for calling a function unless there is a specific readability
problem. Some find that formatting with strictly one argument on each
line is more readable and simplifies editing of the arguments. However,
we prioritize for the reader over the ease of editing arguments, and
most readability problems are better addressed with the following
techniques.

If having multiple arguments in a single line decreases readability due
to the complexity or confusing nature of the expressions that make up
some arguments, try creating variables that capture those arguments in a
descriptive name::

        int my\_heuristic = scores[x] \* y + bases[x];

        bool retval = DoSomething(my\_heuristic, x, y, z);

Or put the confusing argument on its own line with an explanatory
comment::

        bool retval = DoSomething(scores[x] \* y + bases[x],  // Score heuristic.
                                  x, y, z);

If there is still a case where one argument is significantly more
readable on its own line, then put it on its own line. The decision
should be specific to the argument which is made more readable rather
than a general policy.

Sometimes arguments form a structure that is important for readability.
In those cases, feel free to format the arguments according to that
structure::

        // Transform the widget by a 3x3 matrix.

        my\_widget.Transform(x1, x2, x3,

                            y1, y2, y3,

                            z1, z2, z3);

Conditionals
------------

Prefer no spaces inside parentheses. The if and else keywords belong on
separate lines.

There are two acceptable formats for a basic conditional statement. One
includes spaces between the parentheses and the condition, and one does
not.

The most common form is without spaces. Either is fine, but be
consistent. If you are modifying a file, use the format that is already
present. If you are writing new code, use the format that the other
files in that directory or project use. If in doubt and you have no
personal preference, do not add the spaces.

::

        if (condition) {  // no spaces inside parentheses

          ...  // 2 space indent.

        } else if (...) {  // The else goes on the same line as the closing brace.

          ...

        } else {

          ...

        }

--------------

Note that in all cases you must have a space between the if and the open
parenthesis. You must also have a space between the close parenthesis
and the curly brace, if you're using one.

::

        if(condition) {   // Bad - space missing after IF.

        if (condition){   // Bad - space missing before {.

        if(condition){    // Doubly bad.

        if (condition) {  // Good - proper space after IF and before {.

Short conditional statements may be written on one line if this enhances
readability. You may use this only when the line is brief and the
statement does not use the else clause.

::

        if (x == kFoo) return new Foo();

        if (x == kBar) return new Bar();

        This is not allowed when the if statement has an else:

        // Not allowed - IF statement on one line when there is an ELSE clause

        if (x) DoThis();

        else DoThat();

In general, curly braces are not required for single-line statements,
but they are allowed if you like them; conditional or loop statements
with complex conditions or statements may be more readable with curly
braces. Some projects require that an if must always always have an
accompanying brace.

::

        if (condition)
          DoSomething();  // 2 space indent.   // NOT ALLOWED!!! bitchy.
        if (condition) {
          DoSomething();  // 2 space indent.
        }

However, if one part of an if-else statement uses curly braces, the
other part must too::

        // Not allowed - curly on IF but not ELSE
        if (condition) {
          foo;
        } else
          bar;

        // Not allowed - curly on ELSE but not IF
        if (condition)
          foo;
        else {
          bar;
        }

        // Curly braces around both IF and ELSE required because
        // one of the clauses used braces.
        if (condition) {
          foo;
        } else {
          bar;
        }

Loops and Switch Statements
---------------------------

Switch statements may use braces for blocks. Annotate non-trivial
fall-through between cases. Braces are optional for single-statement
loops. Empty loop bodies should use {} or continue.

case blocks in switch statements can have curly braces or not, depending
on your preference. If you do include curly braces they should be placed
as shown below.

If not conditional on an enumerated value, switch statements should
always have a default case (in the case of an enumerated value, the
compiler will warn you if any values are not handled). If the default
case should never execute, simply assert::

        switch (var) {
          case 0: {  // 2 space indent
            ...      // 4 space indent
            break;
          }

          case 1: {
            ...
            break;
          }

          default: {
            assert(false);
          }
        }

Braces are NOT(!) optional for single-statement loops.

::

        for (int i = 0; i < kSomeNumber; ++i)

          printf("I love you\\n");

        for (int i = 0; i < kSomeNumber; ++i) {

          printf("I take it back\\n");

        }

Empty loop bodies should use {} or continue, but not a single semicolon.

::

        while (condition) {
          // Repeat test until it returns false.
        }

        for (int i = 0; i < kSomeNumber; ++i) {}  // Good - empty body.

        while (condition) continue; // Bad - continue indicates no logic.

        while (condition);  // Bad - looks like part of do/while loop.

Pointer and Reference Expressions
---------------------------------

No spaces around period or arrow. Pointer operators do not have trailing
spaces.

The following are examples of correctly-formatted pointer and reference
expressions::

        x = \*p;

        p = &x;

        x = r.y;

        x = r->y;

Note that:

-  There are no spaces around the period or arrow when accessing a
   member.
-  Pointer operators have no space after the \* or &.

When declaring a pointer variable or argument, you may place the
asterisk adjacent to either the type or to the variable name::

        // These are fine, space preceding.

        char \*c;

        const string &str;

        // These are not fine, space following.

        char\* c;    // but remember to do "char\* c, \*d, \*e, ...;"!

        const string& str;

        char \* c;  // Bad - spaces on both sides of \*

        const string & str;  // Bad - spaces on both sides of &

You should do this consistently within a single file, so, when modifying
an existing file, use the style in that file.

Boolean Expressions
-------------------

When you have a boolean expression that is longer than the standard line
length, be consistent in how you break up the lines.

In this example, the logical AND operator is always at the end of the
lines::

        if (this\_one\_thing > this\_other\_thing &&
            a\_third\_thing == a\_fourth\_thing &&
            yet\_another && last\_one) {
          ...
        }

Note that when the code wraps in this example, both of the && logical
AND operators are at the end of the line. This is more common in Google
code, though wrapping all operators at the beginning of the line is also
allowed. Feel free to insert extra parentheses judiciously because they
can be very helpful in increasing readability when used appropriately.
Also note that you should always use the punctuation operators, such
as && and ~, rather than the word operators, such as and and compl.

Return Values
-------------

Do not needlessly surround the return expression with parentheses.

Use parentheses in return expr; only where you would use them in x =
expr;.

::

        return result;                  // No parentheses in the simple case.

        // Parentheses OK to make a complex expression more readable.
        return (some\_long\_condition &&
                another\_condition);

        return (value);                // You wouldn't write var = (value);

        return(result);                // return is not a function!

Variable and Array Initialization
---------------------------------

Your choice of =, (), or {}.

You may choose between =, (), and {}; the following are all correct::

        int x = 3;

        int x(3);

        int x{3};

        string name = "Some Name";

        string name("Some Name");

        string name{"Some Name"};

Be careful when using a braced initialization list {...} on a type with
an std::initializer\_list constructor. A
nonempty braced-init-list prefers the std::initializer\_list constructor
whenever possible. Note that empty braces {}are special, and will call a
default constructor if available. To force the
non-std::initializer\_list constructor, use parentheses instead of
braces.

::

        vector<int> v(100, 1);  // A vector of 100 1s.

        vector<int> v{100, 1};  // A vector of 100, 1.

Also, the brace form prevents narrowing of integral types. This can
prevent some types of programming errors.

::

        int pi(3.14);  // OK -- pi == 3.

        int pi{3.14};  // Compile error: narrowing conversion.

Preprocessor Directives
-----------------------

The hash mark that starts a preprocessor directive should always be at
the beginning of the line.

Even when preprocessor directives are within the body of indented code,
the directives should start at the beginning of the line.

::

        // Good - directives at beginning of line

          if (lopsided\_score) {

        #if DISASTER\_PENDING      // Correct -- Starts at beginning of line

            DropEverything();

        #if NOTIFY                   NotifyClient();

        #endif /\* NOTIFY \*/

        #endif /\* DISASTER\_PENDING \*/

            BackToNormal();

          }

        // Bad - indented directives

          if (lopsided\_score) {

            #if DISASTER\_PENDING  // Wrong!  The "#if" should be at beginning
        of line

            DropEverything();

            #endif                // Wrong!  Do not indent "#endif"

            BackToNormal();

          }

Class Format
------------

Sections in public, protected and private order, each indented by no
space.

The basic format for a class declaration (lacking the comments,
see `Class
Comments <https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Class_Comments>`__ for
a discussion of what comments are needed) is::

        class MyClass : public OtherClass {

        public:

          MyClass();  // Regular 2 space indent.

          explicit MyClass(int var);

          ~MyClass() {}

          void SomeFunction();

          void SomeFunctionThatDoesNothing() {

          }

          void set\_some\_var(int var) { some\_var\_ = var; }

          int some\_var() const { return some\_var\_; }

        private:

          bool SomeInternalFunction();

          int some\_var\_;

          int some\_other\_var\_;

        };

Things to note:

-  Any base class name should be on the same line as the subclass name,
   subject to the 80-column limit.
-  
-  Except for the first instance, these keywords should be preceded by a
   blank line. This rule is optional in small classes.
-  Do not leave a blank line after these keywords.
-  The public section should be first, followed by the protected and
   finally the private section.
-  See Declaration Order for rules on ordering declarations within each
   of these sections.

Constructor Initializer Lists
-----------------------------

Constructor initializer lists can be all on one line or with subsequent
lines indented four spaces.

There are two acceptable formats for initializer lists::

        // When it all fits on one line:

        MyClass::MyClass(int var) : some\_var\_(var), some\_other\_var\_(var + 1)

        {

        }

or

::

        // When it requires multiple lines, indent 4 spaces, putting the colon on
        // the first initializer line:

        MyClass::MyClass(int var)

        :some\_var\_(var)

        ,some\_other\_var\_(var + 1)

        {  // lined up

          ...

          DoSomething();

          ...

        }

Horizontal Whitespace
---------------------

Use of horizontal whitespace depends on location. Never put trailing
whitespace at the end of a line.

General
~~~~~~~

::

        void f(bool b) {  // Open braces should always have a space before them.

          ...

        int i = 0;  // Semicolons usually have no space before them.

        // Spaces inside braces for braced-init-list are optional.  If you use them,
        // put them on both sides!

        int x[] = { 0 };

        int x[] = {0};

        // Spaces around the colon in inheritance and initializer lists.

        class Foo : public Bar {

         public:

          // For inline function implementations, put spaces between the braces

          // and the implementation itself.

          Foo(int b) : Bar(), baz\_(b) {}  // No spaces inside empty braces.

          void Reset() { baz\_ = 0; }  // Spaces separating braces from
        implementation.

          ...

Adding trailing whitespace can cause extra work for others editing the
same file, when they merge, as can removing existing trailing
whitespace. So: Don't introduce trailing whitespace. Remove it if you're
already changing that line, or do it in a separate clean-up operation
(preferably when no-one else is working on the file).

Loops and Conditionals
~~~~~~~~~~~~~~~~~~~~~~

::

        if (b) {          // Space after the keyword in conditions and loops.

        } else {          // Spaces around else.

        }

        while (test) {}   // There is usually no space inside parentheses.

        switch (i) {

        for (int i = 0; i < 5; ++i) {

        // Loops and conditions may have spaces inside parentheses, but this

        // is rare.  Be consistent.

        switch ( i ) {

        if ( test ) {

        for ( int i = 0; i < 5; ++i ) {

        // For loops always have a space after the semicolon.  They may have a
        space

        // before the semicolon, but this is rare.

        for ( ; i < 5 ; ++i) {

          ...

        // Range-based for loops always have a space before and after the colon.

        for (auto x : counts) {

          ...

        }

        switch (i) {

          case 1:         // No space before colon in a switch case.

            ...

          case 2:

            break;

Operators
~~~~~~~~~

::

        // Assignment operators always have spaces around them.
        x = 0;

        // Other binary operators usually have spaces around them, but it's
        // OK to remove spaces around factors.  Parentheses should have no
        // internal padding.
        v = w \* x + y / z;
        v = w\*x + y/z;
        v = w \* (x + z);

        // No spaces separating unary operators and their arguments.
        x = -5;
        ++x;
        if (x && !y)
          ...

Templates and Casts
~~~~~~~~~~~~~~~~~~~

::

        // No spaces inside the angle brackets (< and >), before
        // <, or between >( in a cast
        vector<string> x;
        y = static\_cast<char\*>(x);

        // Spaces between type and pointer are OK, but be consistent.
        vector<char \*> x;
        set<list<string>> x;        // Permitted in C++11 code. bad bad code
        set< list<string> > x;       // C++03 required a space in > >.

        // You may optionally use symmetric spacing in < <.
        set< list<string> > x;

.. |image0| image:: images/image00.gif
