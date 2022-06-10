## sample 2

> Shows a more complex unit test for a class with multiple member functions

This sample shows how to write a more complex unit test for a class that has multiple member functions.

Usually, it's a good idea to have one test for each method in your class. You don't have to do that exactly, but it helps to keep your tests organized. 

The `CMakeLists.txt` is similar to the `sample-1` one.

For testing we're using the following new macro:

- `EXPECT_STREQ()`: expect 2 strings to equal
