## sample 7

> Teaches the basics of value-parametrized tests

This sample shows how to test common properties of multiple implementations of an interface (aka interface tests) using value-parametrized tests. Each test in the test case has a parameter that is an interface pointer to an implementation tested.

```cpp
using ::testing::TestWithParam;
using ::testing::Values;
```

As a general rule, to prevent a test from affecting the tests that come after it, you should create and destroy the tested objects for each test instead of reusing them. In this sample we will define a simple factory function for PrimeTable objects. We will instantiate objects in test's `SetUp()` method and delete them in `TearDown()` method.

```cpp
typedef PrimeTable* CreatePrimeTableFunc();

PrimeTable* CreateOnTheFlyPrimeTable() { return new OnTheFlyPrimeTable(); }

template <size_t max_precalculated>
PrimeTable* CreatePreCalculatedPrimeTable() {
  return new PreCalculatedPrimeTable(max_precalculated);
}
```

Inside the test body, fixture constructor, `SetUp()` and `TearDown()` you can refer to the test parameter by `GetParam()`. In this case, the test parameter is a factory function which we call in fixture's `SetUp()` to create and store an instance of `PrimeTable`.

```cpp
class PrimeTableTestSmpl7 : public TestWithParam<CreatePrimeTableFunc*> {
 public:
  ~PrimeTableTestSmpl7() override { delete table_; }
  void SetUp() override { table_ = (*GetParam())(); }
  void TearDown() override {
    delete table_;
    table_ = nullptr;
  }

 protected:
  PrimeTable* table_;
};
```

```cpp
TEST_P(PrimeTableTestSmpl7, ReturnsFalseForNonPrimes) {
  EXPECT_FALSE(table_->IsPrime(-5));
  EXPECT_FALSE(table_->IsPrime(0));
  EXPECT_FALSE(table_->IsPrime(1));
  EXPECT_FALSE(table_->IsPrime(4));
  EXPECT_FALSE(table_->IsPrime(6));
  EXPECT_FALSE(table_->IsPrime(100));
}
```

In order to run value-parametrized tests, you need to instantiate them, or bind them to a list of values which will be used as test parameters. 

Here, we instantiate our tests with a list of two PrimeTable object factory functions:

```cpp
INSTANTIATE_TEST_SUITE_P(OnTheFlyAndPreCalculated, PrimeTableTestSmpl7,
                         Values(&CreateOnTheFlyPrimeTable,
                                &CreatePreCalculatedPrimeTable<1000>));
```

