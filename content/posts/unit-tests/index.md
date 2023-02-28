---
title: "Why Data Scientists Should Write Unit Tests for Their Code"
date: 2023-02-28T13:58:41+01:00
draft: false

cover:
    image: "images/unit-tests.png"
    alt: "unit tests"
    caption: "unit testing"
    relative: true

---

Within the software engineering industry most developers will be familiar with **unit testing**. A unit test aims to check whether a part of your code operates in the intended way. Writing them has the following benefits:

* Reduces bugs when developing new features or when changing the existing functionality

* Prevents unexpected output

* Helps detecting edge cases

* Tests can serve as documentation

> All the benefits of unit testing for software engineering projects apply to data science projects as well

![Meme](images/meme.jpeg)

When should you start writing tests? The more certainty you want to have that your code works as intended, the more you want to invest in testing. When you are exploring a dataset for the first time, you will be less inclined to write them. But once you move beyond the exploratory phase, you should start adding some tests. Especially when code is used in production, writing tests can be very valuable and ultimately time-saving. I would like to showcase the need for it using a very practical example.

## Coding without testing

Let’s say you have a dataframe with a text column that contains monetary values, e.g. “This product costs 5 euro”. Your goal is to write a function that extracts the value 5. This would look like something as follows:

```python
import re
import pandas as pd

df = pd.DataFrame(
    {
        "text": ["5 euro", "7 euro"],
        "row_number": [1, 2],
    }
)


def extract_money(text):
    """Extract monetary value from string by looking for
    a pattern of a digit, followed by 'euro'.
    e.g. 5 euro --> 5
    Args:
        text (str): Text containing monetary value
    Returns:
        float: The extracted value
    """

    extracted_money = re.search(r"(\d) euro", text).group(1)
    return float(extracted_money)


df["money"] = df["text"].apply(lambda x: extract_money(x))

```

This works well for your current dataset, printing the following table:
```text
    +--------+------------+--------+
    | text   | row_number | money  |
    +--------+------------+--------+
    | 5 euro |          1 |    5.0 |
    | 7 euro |          2 |    7.0 |
    +--------+------------+--------+
```
You are happy with this result and proud of yourself that you wrote some docstrings, so you incorporate the code into your pipeline and go on to your next task. The next week your manager comes back, furious. “Our application stopped working when new data came in, this is costing the business millions of $”, he exclaims. You anxiously go back to check out the new dataset and see what has gone wrong:

```python
df = pd.DataFrame(
    {
        "text": ["5 euro", "7 euro", ""],
        "row_number": [1, 2, 3],
    }
)
```

> An empty string has entered the chat, resulting in the following error: *AttributeError: ‘NoneType’ object has no attribute ‘group’*

You realize that you have to change the function to account for the scenario of the string being empty and make the relevant change to the code.

```python
def extract_money(text):
    """Extract monetary value from string by looking for
    a pattern of a digit, followed by 'euro'.
    e.g. 5 euro --> 5
    Args:
        text (str): Text containing monetary value
    Returns:
        float: The extracted value
    """

    if text:
        extracted_money = re.search("(\d) euro", text).group(1)
        return float(extracted_money)
    else:
        return None
```

However, you also come to the realization that you should have thought ahead, and that more unanticipated rows of data might appear in the future. You want to be fully confident that your code is working exactly as you intended.

## Unit Tests to the rescue

This is the mindset that writing unit tests encourages. **Pytest** is a great tool for writing unit tests in Python. It makes writing tests short, easy to read, and provides great output. You can install pytest using pip install pytestand create a folder structure like the following:

    project/
    ├─ src/
    │  ├─ regex.py
    │  ├─ __init__.py
    ├─ test/
    │  ├─ test_regex.py
    │  ├─ __init__.py

Within text_regex.py I import my extract_money function and write a first test. The test consists of the AAA (Arrange-Act-Assert) pattern. Some data is created (Arrange), in this case an empty string. We then invoke the method that we want to test (Act). Lastly, we check whether the outcome of the function matches the expected output (Assert).

```python
from src.regex import extract_money
import pytest


def test_empty_string():
    empty_string = ""
    extracted_money = extract_money(empty_string)
    expected_output = None
    assert extracted_money == expected_output
```
>  Note: both the filename and the name of a test function itself should always start with *test_* , otherwise the function will not be detected by *pytest* .

To check whether the function can now indeed handle an empty string I run the test from the root directory using pytest -v , which returns the following output:

![Meme](images/output.png)

Yeah, the test has passed! The change that we made before gives us the expected result. Alright, what other cases can we think of? Lets try and see what happens when there is a monetary value with decimals, like “5.49 euro”.

```python
from src.regex import extract_money
import pytest


def test_empty_string():
    empty_string = ""
    extracted_money = extract_money(empty_string)
    expected_output = None
    assert extracted_money == expected_output


def test_money_with_decimal_numbers():
    decimal_string = "5.49 euro"
    extracted_money = extract_money(decimal_string)
    assert extracted_money == 5.49
```

Again we run pytest -v , which gives us the following output:

![Meme](images/failed-test.png)

As you can see, we got an AssertionError, stating that 9 is not equal to 5.49. It seems that our function only extracted the last digit of 5.49. Without this test, we would have been less likely to catch this error. The function would have returned the value 9, instead of throwing us an error. Good thing that we found this bug before the manager did.

>  An important goal of testing within data science is to prevent unexpected output from happening

The next step is to fix our code and make sure that it now passes the test that failed before, while still passing the first test that we wrote as well. A great thing about unit testing is that it gives you an overview of **how your changes affect the project as a whole**. The tests will make you aware if any of your code additions cause any unexpected consequences.

Furthermore, the fact that our initial function did not pass the decimal number test raises the question what other examples might come up that we did not anticipate. Maybe we will get a number separated by a comma, like 5,49 euro. Or the monetary value is formatted as € 5.49,-. Unit testing allows us to quickly check for these cases and some other less-common **edge cases**.

## Documentation
You could argue that for this example you could use a site like [regex101](https://regex101.com) to test your regex. However, one of the advantages of unit tests that I highlighted in the introduction is that the tests can serve as **documentation**. Test functions illustrate which scenarios you thought of when developing a functionality and give concrete examples of these scenarios. When onboarding a new colleague to your project it becomes immediately clear to them what a given function is supposed to do, just by looking at the test functions. But also when coming back to the project yourself after a couple of months the tests will help you remember what you were trying to accomplish with your code.

## Thank you for reading!

I hope that this article has given you a better understanding about why testing is a good idea for your data science projects.

I encourage you to use this code as a basis and try to write some more tests for this function. The **code** used in this article can be found [here](https://github.com/bjornvandijkman-vantage/unit_testing_tutorial). In a next article on testing I would like to focus on more advanced functionalities of Pytest like the **fixture** and **parametrize decorators**. Feel free to add me on [LinkedIn](https://www.linkedin.com/in/björn-van-dijkman-5103a671/) if you are, like me, passionate about data science and the engineering side of it.

--- 

*Thank you to my colleagues Ruurtjan Pul, Max Ortega del Vecchyo, Menno Liefstingh and Karlijn Schipper for reviewing this blog*
