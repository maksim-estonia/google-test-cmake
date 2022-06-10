## sample 6

> Demonstrates type-parametrized tests

This sample shows how to test common properties of multiple implementations of the same inferface (aka interface tests).

First, we define some factory functions for creating instances of the implementations. You may be able to skip this step if all your implementations can be constructed the same way.

```cpp
template <class T>
PrimeTable* CreatePrimeTable();

template <>
PrimeTable* CreatePrimeTable<OnTheFlyPrimeTable>() {
  return new OnTheFlyPrimeTable;
}

template <>
PrimeTable* CreatePrimeTable<PreCalculatedPrimeTable>() {
  return new PreCalculatedPrimeTable(10000);
}
```

Then we define a test fixture class template.

```cpp
template <class T>
class PrimeTableTest : public testing::Test {
 protected:
  // The ctor calls the factory function to create a prime table
  // implemented by T.
  PrimeTableTest() : table_(CreatePrimeTable<T>()) {}

  ~PrimeTableTest() override { delete table_; }

  // Note that we test an implementation via the base interface
  // instead of the actual implementation class.  This is important
  // for keeping the tests close to the real world scenario, where the
  // implementation is invoked via the base interface.  It avoids
  // got-yas where the implementation class has a method that shadows
  // a method with the same name (but slightly different argument
  // types) in the base interface, for example.
  PrimeTable* const table_;
};
```

```cpp
using testing::Types;
```

Google Test offers two ways for reusing tests for different types. The first is called "typed tests". You should use if if you already know all the types you are gonna exercise when you write the tests.

To write a typed test case, first use

```cpp
TYPED_TEST_SUITE(TestCaseName, TypeList);
```

to declare it and specify the type parameters. As with TEST_F, `TestCaseName` must match the test fixture name.

The list of types we want to test:

```cpp
typedef Types<OnTheFlyPrimeTable, PreCalculatedPrimeTable> Implementations;

TYPED_TEST_SUITE(PrimeTableTest, Implementations);
```

Then use `TYPED_TEST(TestCaseName, TestName)` to define a typed test, similar to `TEST_F`.

```cpp
TYPED_TEST(PrimeTableTest, ReturnsFalseForNonPrimes) {
  // Inside the test body, you can refer to the type parameter by
  // TypeParam, and refer to the fixture class by TestFixture.  We
  // don't need them in this example.

  // Since we are in the template world, C++ requires explicitly
  // writing 'this->' when referring to members of the fixture class.
  // This is something you have to learn to live with.
  EXPECT_FALSE(this->table_->IsPrime(-5));
  EXPECT_FALSE(this->table_->IsPrime(0));
  EXPECT_FALSE(this->table_->IsPrime(1));
  EXPECT_FALSE(this->table_->IsPrime(4));
  EXPECT_FALSE(this->table_->IsPrime(6));
  EXPECT_FALSE(this->table_->IsPrime(100));
}
```

That's it! Google Test will repeat each `TYPED_TEST` for each type in the type list specified in `TYPED_TEST_SUITE`. Now you don't have to define them multiple times.

---

Sometimes, however, you don't yet know all the types that you want to test when you write the tests to make sure each implementation conforms to some basic requirements, but you don't know what implementations will be written in the future.

How can you write the tests without comitting to the type parameters? That's what "type-parametrized tests" can do for you. It is a bit more involved than typed tests, but in return you get test pattern that can be reused in many contexts, which is a big win.

First, define a test fixture class template. Here we just reuse the `PrimeTableTest` fixture defined earlier:

```cpp
template <class T>
class PrimeTableTest2 : public PrimeTableTest<T> {};
```

Then, declare the test case. The argument is the name of the test fixture, and also the name of the test case (as usual). The `_P` suffix is for parametrized or pattern.

```cpp
TYPED_TEST_SUITE_P(PrimeTableTest2);
```

Next, use `TYPED_TEST_P(TestCaseName, TestName)` to define a test, similar to what you do with `TEST_F`.

```cpp
TYPED_TEST_P(PrimeTableTest2, ReturnsFalseForNonPrimes) {
  EXPECT_FALSE(this->table_->IsPrime(-5));
  EXPECT_FALSE(this->table_->IsPrime(0));
  EXPECT_FALSE(this->table_->IsPrime(1));
  EXPECT_FALSE(this->table_->IsPrime(4));
  EXPECT_FALSE(this->table_->IsPrime(6));
  EXPECT_FALSE(this->table_->IsPrime(100));
}
```

Type-parametrized tests involve one extra step: you have to enumerate the tests you defined:

```cpp
REGISTER_TYPED_TEST_SUITE_P(
    PrimeTableTest2,  // The first argument is the test case name.
    // The rest of the arguments are the test names.
    ReturnsFalseForNonPrimes, ReturnsTrueForPrimes, CanGetNextPrime);
```

At this point the test pattern is done. However, you don't have any real test yet as you haven't said which types you want to run the tests with.

To turn the abstract test pattern into real tests, you instantiate it with a list of types. Usually the test pattern will be defined in a `.h`, and anyone can `#include` and instantiate it. 

```cpp
// The list of types we want to test.  Note that it doesn't have to be
// defined at the time we write the TYPED_TEST_P()s.
typedef Types<OnTheFlyPrimeTable, PreCalculatedPrimeTable>
    PrimeTableImplementations;
INSTANTIATE_TYPED_TEST_SUITE_P(OnTheFlyAndPreCalculated,    // Instance name
                               PrimeTableTest2,             // Test case name
                               PrimeTableImplementations);  // Type list
```

