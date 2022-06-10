## sample 5

> Puts shared testing logic in a base test fixture, and reuses it in derived fixtures

When you define a test fixture, you specify the name of the test case that will use this fixture. Therefore, a test fixture can be used by only one test case.

Sometimes, more than one test case may want to use the same or slightly different test fixtures. In Google Test, you do this by putting the shared logic in a super (class) test fixture, and then have each test case use fixture derived from this super fixture.

In this sample, we want to ensure that every test finishes within ~5 seconds. If a test takes longer to run, we consider it a failure.

We put the code for timing a test in a test fixture called "Quicktest". QuickTest is intended to be the super fixture that other fixtures derive from, therefore there is no test case with the name "QuickTest".

```cpp
class QuickTest : public testing::Test {
 protected:
  // Remember that SetUp() is run immediately before a test starts.
  // This is a good place to record the start time.
  void SetUp() override { start_time_ = time(nullptr); }

  // TearDown() is invoked immediately after a test finishes.  Here we
  // check if the test was too slow.
  void TearDown() override {
    // Gets the time when the test finishes
    const time_t end_time = time(nullptr);

    // Asserts that the test took no more than ~5 seconds.  Did you
    // know that you can use assertions in SetUp() and TearDown() as
    // well?
    EXPECT_TRUE(end_time - start_time_ <= 5) << "The test took too long.";
  }

  // The UTC time (in seconds) when the test starts
  time_t start_time_;
};
```

We derive a fixture named IntegerFunctionTest from the QuickTest fixture. All tests using this fixture will be automatically registered to be quick.

```cpp
class IntegerFunctionTest : public QuickTest {
  // We don't need any more logic than already in the QuickTest fixture.
  // Therefore the body is empty.
};
```

Now we can write tests in the IntegerFunctionTest test case.

```cpp
// Tests Factorial()
TEST_F(IntegerFunctionTest, Factorial) {
  // Tests factorial of negative numbers.
  EXPECT_EQ(1, Factorial(-5));
  EXPECT_EQ(1, Factorial(-1));
  EXPECT_GT(Factorial(-10), 0);

  // Tests factorial of 0.
  EXPECT_EQ(1, Factorial(0));

  // Tests factorial of positive numbers.
  EXPECT_EQ(1, Factorial(1));
  EXPECT_EQ(2, Factorial(2));
  EXPECT_EQ(6, Factorial(3));
  EXPECT_EQ(40320, Factorial(8));
}
```

