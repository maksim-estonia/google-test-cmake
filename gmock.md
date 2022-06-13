## gMock

When you write a prototype or test, often it's not feasible or wise to rely on real objects entirely. A mock object implements the same interface as a real object (so it can be used as one), but let's you specify at run time how it will be used and what is should do (which methods will be called? in which order? how many times? with what arguments? what will they return? etc)

Mocks are objects pre-programmed with _expectations_, which form a specification of the calls they are expected to receive.

When using gMock,

1. First, you use some simple macros to describe the interface you want to mock, and they will expand to the implementation of your mock class.
2. Next, you create some mock objects and specify its expectations and behaviour using an intuitive syntax.
3. Then you exercise code that uses the mock objects. gMock will catch any violations to the expectations as soon as it arises.

### sample 1: turtle

Writing the Mock Class

How to define it

Using the `Turtle` interface as example, here are the simple steps you need to follow:

- Derive a class `MockTurtle` from `Turtle`
- Take a _virtual_ function of `Turtle` 
- In the `public:` section of the child class, write `MOCK_METHOD();`
- Take the function signature, cut-and-paste it into the macro, and add two commas - one between the return type and the name, another between the name and the argument list.
- If you're mocking a const method, add a 4th parameter containing `(const)` 
- Since you're overriding a virtual method, we suggest adding the `override` keyword. For const methods the 4th parameter becomes `(const, override)`.
- Repeat until all virtual function you wnat to mock are done. (It goes without saying that all pure virtual methods in your abstract class must be either mocked or overridden.)

After the process, you should have something like:

```cpp
#include "gmock/gmock.h"  // Brings in gMock.

class MockTurtle : public Turtle {
 public:
  ...
  MOCK_METHOD(void, PenUp, (), (override));
  MOCK_METHOD(void, PenDown, (), (override));
  MOCK_METHOD(void, Forward, (int distance), (override));
  MOCK_METHOD(void, Turn, (int degrees), (override));
  MOCK_METHOD(void, GoTo, (int x, int y), (override));
  MOCK_METHOD(int, GetX, (), (const, override));
  MOCK_METHOD(int, GetY, (), (const, override));
};
```

Using Mocks in Tests

Once you have a mock class, using it is easy. The typical work flow:

1. Import the gMock names from the `testing` namespace such that you can use them unqualified. 
2. Create some mock objects.
3. Specify your expectation on them. (How many times will a method be called? With what arguments? What should it do?)
4. Exercise some code that uses the mocks; optionally, check the result using googletest assertions. If a mock is called more than expected or with wrong arguments, you'll get an error immediately.
5. When a mock is destructed, gMock will automatically check whether all expectations on it have been satisfied.
   
```cpp
#include "path/to/mock-turtle.h"
#include "gmock/gmock.h"
#include "gtest/gtest.h"

using ::testing::AtLeast;                         // #1

TEST(PainterTest, CanDrawSomething) {
  MockTurtle turtle;                              // #2
  EXPECT_CALL(turtle, PenDown())                  // #3
      .Times(AtLeast(1));

  Painter painter(&turtle);                       // #4

  EXPECT_TRUE(painter.DrawCircle(0, 0, 10));      // #5
}
```

Note: gMock requires expectations to be set before the mock functions are called, otherwise the behaviour is undefined. Do not alternate between calls to `EXPECT_CALL()` and calls to the mock functions, and do not set any expectations on a mock after passing the mock to an API.

### Setting Expectations

In gMock we use the `EXPECT_CALL()` macro to set an expectation on a mock method. The general syntax is:

```cpp
EXPECT_CALL(mock_object, method(matchers))
    .Times(cardinality)
    .WillOnce(action)
    .WillRepeatedly(action);
```

The macro has two arguments: first the mock object, and then the method and its arguments. 

Matchers: What arguments do we expect?

When a mock function takes arguments, we may specify what arguments we are expecting, for example:

```cpp
// Expects the turtle to move forward by 100 units.
EXPECT_CALL(turtle, Forward(100));
```

Often you do not want to be too specific. If you aren't interested in the value of an argument, write `_` as the argument, which means "anything goes".

```cpp
using ::testing::_;
...
// Expects that the turtle jumps to somewhere on the x=50 line.
EXPECT_CALL(turtle, GoTo(50, _));
```

`_` is an instance of what we call matchers. A matcher is like a predicate and can test whether an argument is what we'd expect. 

If you don't care about _any_ arguments, rather than specify `_` for each of them you may instead omit the parameter list.

```cpp
// Expects the turtle to move forward.
EXPECT_CALL(turtle, Forward);
// Expects the turtle to jump somewhere.
EXPECT_CALL(turtle, GoTo);
```

Cardinalities: how many times will it be called?

Actions: what should it do?

Remember that mock object doesn't really have working implementation? We as users have to tell it what to do when a method is invoked. This is easy in gMock.

First, if the return type of a mock function is a built-in type or a pointer, the function has a default action (a void function will just return, a bool function will return false, and other functions will return 0). 

Second, if a mock function doesn't have a default action, or the default action doesn't suit you, you can specify the action to be take each time the expectation matches uing a series of `WillOnce()` clauses followed by an optional `WillRepeatedly()`.

```cpp
using ::testing::Return;
...
EXPECT_CALL(turtle, GetX())
     .WillOnce(Return(100))
     .WillOnce(Return(200))
     .WillOnce(Return(300));
```

Says that `turtle.getX()` will be called exactly three times (gMock inferred this from how many `WillOnce()` clauses we've written, since we didn't explicitly write `Times()`), and will return 100, 200 and 300 respectively.

```cpp
using ::testing::Return;
...
EXPECT_CALL(turtle, GetY())
     .WillOnce(Return(100))
     .WillOnce(Return(200))
     .WillRepeatedly(Return(300));
```

says that `turtle.GetY()` will be called at least twice. Will return 100 and 200 respectively the first two times, and 300 from the third time on.

What can we do inside `WillOnce()` besides `Return()`? You can return a reference using `ReturnRef()`, or invoke a pre-defined function, among [others](https://google.github.io/googletest/gmock_cook_book.html#using-actions).

Using Multiple Expectations

By default, when a mock method is invoked, gMock will search the expectations in the reverse order they are defined, and stop when an active expectation that matches the arguments is found (you can think of it as newer rules override older rules). 