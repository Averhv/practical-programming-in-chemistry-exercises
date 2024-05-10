## Setup

Go to https://github.com/schwallergroup/Rxn-INSIGHT and read the README.

Follow the installation instructions, making sure to install the optional dependencies
as these will be required to run the tests and style checks.

Make sure you are in the top directory, and in your newly-created conda environment
`rxn-insight`, then run the tests with the `tox` command.


## Writing a failing test, then fixing it

Run the test environment using the command `tox -e py3` and inspect the terminal output.
First, tox builds the correct test environment with the required dependencies for
running the tests. It is told how to do so in the `tox.ini` file. Then, it tests the
code using `pytest`. You can see this in the `tox.ini` file, under the `[testenv]`
block:
```
commands =
    mypy src tests
    pytest {env:PYTEST_MARKERS:} {env:PYTEST_EXTRA_ARGS:} {posargs:-vv}
    coverage: genbadge coverage -i coverage.xml    
```
Essentially this looks for any file named `test_{...}.py` in the
`Rxn-insight/tests/` directory and executes the code within them. You should see an
output in the terminal like this:

```
tests/test_classification.py::test_initialization PASSED
tests/test_classification.py::test_get_template_smiles PASSED
tests/test_import.py::test_import PASSED
```

Open the `tests/test_classification.py` file and inspect the tests. What is the code
doing and why do the tests pass?

In the case of the function `test_initialization` in `test_classification.py`, a
`ReactionClassifier` object is initialized with a reaction SMILES string. If the code
runs without error, the test is considered *passed*.

What if we pass an invalid SMILES string? A good test could be that an appropriate error
message is raised. 

To the file `test_classification.py`, add `import pytest` to the top of the file, as the
first import. Copy the following unit test into the test file:

```py
def test_initialization_error():
    """
    Tests that the appropriate error is raised when initializing the
    ReactionClassifier class with an invalid reaction SMILES
    """
    rxn_smiles_with_atom_mapping = "invalid_smiles"

    ReactionClassifier(rxn_smiles_with_atom_mapping, keep_mapping=True)
```

and re-run the tests.

Inspect the output. The test fails because the code raises an error. This is what we
want: passing the string `"invalid_smiles"` as the reaction SMILES should raise an
error. However the test should still pass, so we need to modify the test to tell pytest
that we **expect** an error. 


Modify the test `test_initialization_error` to ***catch*** the
exception with the `pytest.raises` context manager:
```py
with pytest.raises(ValueError) as e:
    ReactionClassifier(rxn_smiles_with_atom_mapping, keep_mapping=True)
assert str(e.value).startswith("Invalid reaction SMILES")
```
Where we check that the appropriate error message is raised by asserting that the
message starts with `"Invalid reaction SMILES"`. If you inspect module
`classification.py` in the source code, you will see in the constructor of the class
(i.e. the `__init__()` method of `ReactionClassifier`) where this error is raised.

## Coverage reports

When writing a series of tests for code, it can be useful to know how much of the code
is 'touched' (or ***covered***) by the tests. Ideally, tests would cover as much of the
relevant source code (i.e. in `Rxn-insight/src/`) as possible to ensure that the code is
working properly.

We can run the tests again, but generate a coverage report in the process to check this.

Run the command `tox -e py3-coverage` and inspect the output. The bit relevant to
coverage here is:
```
Name                                Stmts   Miss  Cover
-------------------------------------------------------
src/rxn_insight/__init__.py             0      0   100%
src/rxn_insight/classification.py    1002    800    20%
src/rxn_insight/reaction.py           318    318     0%
src/rxn_insight/representation.py      22     22     0%
src/rxn_insight/utils.py              405    243    40%
-------------------------------------------------------
TOTAL                                1747   1383    21%
Coverage XML written to file coverage.xml
```

This tells us how much of each file in `src/` is covered by the tests. As we can see,
the tests that have been provided cover some of the code in the `classification.py`
module (and by
extension `utils.py`, as these utility functions are used in `classification.py`), but
not the other modules.


### A note on test-driven development

In our case, we are retrospectively writing tests for code that has already been
written. However, when building your owm project you can use the principle of
***'test-driven development'*** (read more here:
https://en.wikipedia.org/wiki/Test-driven_development) to help you design your codebase.
In essence, you can write a series of tests that capture the functionality of your
package from the user's perspective, including tests that check the code runs as
expected, with expected outputs and without error, as well as those that run with
expected errors. Then you write the actual source code such that all the tests pass.
This can be useful in writing well-designed software, and naturally leads you to write
code that has a high coverage.

**A caveat to coverage**: something to be aware of, for your own code and for others', is
that a high coverage score doesn't necessarily mean the code is well tested. Tests could
cover a large part of the codebase by just running class methods, without testing the
outputs of these methods. Tests are a tool to build well-designed code, and a high
coverage score should be a consequence of well-designed tests instead of a metric to be
maximised in isolation.


# Main exercise

Now for the main exercise of today. This is purposefully left open-ended so you
have space to think about code design, functionality, and user experience. 

**The aim is simple**: get the code coverage as high as possible, ideally > 80%, by
writing a series of ***well-designed*** tests.

Some general advice to help you in the process:

* Each python module has its own test module. For instance, currently we have
  `tests/test_classification.py` for the module `src/rxn_insight/classification.py`.
* Each unit test (i.e. the test function, such as `test_initialization` in
  `test_classification.py`) should be short and test **one aspect** of the
  functionality. If a test fails, it should be easy to debug as it ideally will not be
  ambiguous what part of the code is being tested.
* Keep unit test names descriptive, and write short docstrings for the tests. This helps
  others (or you later on) read, improve, and add to your tests more easily.
* Make sure you understand the functionality of the package you are trying to test. Open
  a notebook in jupyter lab and play around with the classes and functions. This will
  help you to understand what should work, and hopefully what shouldn't, and get you
  thinking about appropriate tests.
* Usually good tests won't just test that code runs without error, but check the output
  of the code too. Conditional checks can be useful for this: i.e. `assert`ing that
  output is equal to some expected output.
* In this exercise, fancy tests aren't required. However, so that you're aware, `pytest`
  has a lot of nice functionality to help write more complex tests without creating a
  lot of messy test code. More can be read here:
  https://docs.pytest.org/en/7.1.x/how-to/index.html . Examples include the use of
  fixtures, parametrization, and doctests.


### Need help getting started? 

First, you can try to increase the coverage of tests for the module `classification.py`.
A good way to do this is to test the class methods, such as
`get_functional_group_smarts`. Remember: we don't just want to test that the code runs,
but that the outputs are as expected.

Next, move on to the other modules. You could start with `reaction.py`:

1. Open the file `reaction.py` and inpect the code. What does the `Reaction` class do?
   What class methods are there and what do they do?
2. Open a jupyter notebook and import the `Reaction` class. Initialize a `Reaction`
   object with the appropriate parameters and try a few of the class methods.
3. Start with writing a test for the initialization of the object, as was done for
   `classification.py`, then move onto testing the class methods.