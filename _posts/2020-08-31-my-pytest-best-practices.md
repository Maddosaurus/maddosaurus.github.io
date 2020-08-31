---
layout: post
title:  "My Python Testing Best Practices"
date:   2020-08-31 16:00 +0100
---
As someone who has been using Python professionally in the younger past, I found some best practices regarding testing and project setup that work well for me. Today I'd like to share them with you.

<!--more-->

**TL;DR**: The repo containing demo code: [Maddosaurus/pytest-practice](https://github.com/Maddosaurus/pytest-practice).


## Project Setup
The project setup is based on Kenneth Reitz' [Hitchhiker's Guide To Python](https://docs.python-guide.org/). It follows the idea of a seperate module accompanied by tests and supporting info on the same level (i.e. not contained in the module itself). This keeps the module lean and small.  
```bash
# Module containing the code
pytdemo/pytdemo.py
pytdemo/util.py
# Testsuite
tests/conftest.py
tests/test_pytdemo.py
tests/test_util.py
# Supporting information
.gitignore
LICENSE
README.md
requirements.txt
setup.py
```
As one can see, the module itself only contains the bare essentials. The test suite is organized to roughly match the submodules, but this is an idea I only use for smaller modules. If submodules get larger and more complex, I tend to group tests by behaviour or logical groups.

## Test Setup
Personally, I'm using a wild mixture of [pytest](https://docs.pytest.org/en/stable/), [unittest.mock.MagicMock](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.MagicMock) and [requests-mock](https://requests-mock.readthedocs.io/en/latest/) - the last one only if the module is using `requests` directly to interact with REST APIs.  
As a general recommendation, you should monitor your test coverage. To do that, I like to use [pytest-cov](https://pytest-cov.readthedocs.io/en/latest/readme.html), which is a powerful tool that can generate nice reports with the `--cov-report html` option.  
A word of caution: Aiming for 100% coverage is a great thing to do, but don't try to enforce it. This can end up being extremely tedious and sometimes impossible. Try to find smart goals instead, i.e. agreeing on covering all functionally important parts of your project.

### The conftest File
You might have noticed that there is a `conftest.py` living in the `tests` folder. This file is used to store shared [pytest fixtures](https://docs.pytest.org/en/stable/fixture.html) that can be used in all test files. This is highly recommended, especially for helper functions and data sources.  
In the example code you will find a fixture here that creates a custom instance of the main module which contains an URL that is pointing to localhost. This is to ensure that even if your mocked endpoints don't catch every call, you'll be the first to know (and also, we're avoiding hitting the real service with test-based requests).

## Patching Monkeys
There is one problem when writing tests: You want to test as small and free of side effects as possible. This can be achieved by mocking away all calls to other functions the subject under test (SUT) is calling. Pytest does this by providing multiple mechanisms with [`monkeypatch`](https://docs.pytest.org/en/stable/monkeypatch.html) being my favourite for its balance between readability and explicitness.  
As an example:
```python
def test_get_all_URL(crt_mock, monkeypatch):
    # Set up a mock that will replace the requests module in CrtSh and use it
    # The MagicMock class comes in handy as it does a lot in the background for us.
    requests_mock = MagicMock()
    monkeypatch.setattr(pytdemo.requests, "get", requests_mock)

    crt_mock.get_all("testhost.domain")

    # Check it the requests module was called with the correct URL
    requests_mock.assert_called_once_with(
        "https://test.local/",
        params={"Identity": "testhost.domain", "output": "json"}
    )
```
In the example the *get* function of `requests` is replaced with a custom `MagicMock` object which will save every call to it.  
As you can see, the test is divided into three parts:  
  1. Arrange - Set up all required vars, data and mocks
  2. Act - Call the SUT
  3. Assert - Checking the result for correctness  

This threefold structure improves readability - conventions often help to make your job as a team easier. It is actually a well known pattern called [AAA - Arrange, Act, Assert](https://freecontent.manning.com/making-better-unit-tests-part-1-the-aaa-pattern/) and I recommend reading a bit more about it if you're interested in writing better Unit Tests.

As you can see, combining different Python tools for testing can yield a very powerful setup that allows you to build your tests quick and easy.

## Final Thoughts
I'm merely scratching the surface with this post. There are many more modules and best practices I'd like to share over time that easen your life as a Python dev.  
For now, keep in mind that testing might feel it is slowing you down but in fact ensures that you keep your current speed. Writing good Unit Tests ensures that all the parts in your module work as intended and keep working as intended - even when you're changing and refactoring the code. This also means that you should run your tests often, so please make sure that they execute as fast as possible.  
Finally, maybe the most important tests you'll write are the ones that are created in the context of a bug ticket:  
Try reproducing the bug with a Unit Test first before attempting to fix it. With this order of operations you ensure that the bug is fixed and that it won't come back at a later stage, so get testing!
