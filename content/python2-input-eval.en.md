---
title: 'In Python 2, input() is eval(raw_input())'
date: '2024-10-06T22:43:58+09:00'
---

## Building the Easyplotlib Structure

The very first step I took while developing [Easyplotlib](../../project/easyplotlib) was to define the communication protocol between the frontend and backend. To briefly explain this project, its main function is to automatically generate code according to the user's requirements (though it's not AI), execute it, and then show the results. Since the project dynamically generates and executes code, I was very mindful of the risk of introducing [ACE (Arbitrary Code Execution)](https://www.okta.com/identity-101/arbitrary-code-execution/) vulnerabilities if the structure was poorly designed.

## Arbitrary Code Execution Vulnerability

Generating code directly in the browser and sending it to the server for execution is far too simple, but anyone with even a basic understanding of security would know to absolutely avoid this development method. The server might expect harmless code like `matplotlib.pyplot.plot()`, but in reality, code like the following could be sent:

```python
import subprocess
subprocess.run(['rm', '-rf', '/', '--no-preserve-root'])
```

### Python 2's input() Is Dangerous

For example, in Python 2, there were two functions for receiving user input from the terminal: `raw_input()` and `input()`. `raw_input()` always returns a str type, just like Python 3's `input()`, but `input()` would automatically cast the input if possible, which made it popular when I first learned Python.

```python
# Python 2
>>> raw_input()
3    # input
'3'  # output
>>> input()
3    # input
3    # output
```

But the shocking truth is that [Python 2's input() implementation](https://python.readthedocs.io/en/v2.7.2/library/functions.html#input) was actually `eval(raw_input())`! The security issues that could arise from this are almost unimaginable.

Another example of ACE is buffer overflow attacks in languages like C or C++ that allow direct access to memory. These can also be considered a type of ACE vulnerability, since the concept is the same: executing code in user input areas that would not normally be executed.

Of course, there are many ways to prevent the above ACE vulnerabilities. In Python, you could use a custom language compiled with file and networking features removed, add a virtualization layer using AWS, or restrict certain functions with pattern matching or external packages like [RestrictedPython](https://github.com/zopefoundation/RestrictedPython). But the best approach is to prevent ACE from the start unless it's absolutely necessary, like in online judges such as [Baekjoon](https://www.acmicpc.net/).

## Easyplotlib Request Design

Actually, Easyplotlib never considered such a problematic design in the first place, since the data could be structured. The only issue was that the structured data was so complex that writing code to validate it from scratch seemed quite tedious.

Click the card below to see a 'rough version' of the data definition. It's not structured as a JSON schema, and since it's taken directly from the design document, it's not very clean.

{{% details title="**View Structured Data (JSON)**" closed="true" %}}

* request_id : *Is uuid4*
* figure
    * size
        * row : *Is numeric, plt.subplot(row, _)*
        * column : *Is numeric, plt.subplot(_, column)*
    * axes [List] : *length of `figure.size.row`*
        * [List] : *length of `figure.size.column`*
            * *Is one of `axes[].name` or null. Null axes will not be rendered, as it never exists*
    * style
        * *Every possible key-value pairs are defined at __figure-style__*
* axes [List]
    * name : *Is string*
    * plot [List]
        * *Is one of `plot[].name`*
    * style
        * *Every possible key-value pairs are defined at __axes-style__*
* plot [List]
    * name : *Is string*
    * format : *Every possible values are defined at __plot-format-list__*
    * data
        * key: *Depending on `plot[].format`, there are different required and optional keys. Check __plot-format-list__.*
        * value: *Is one of `data[].name`*
    * style
        * *Every possible key-value pairs are defined at __plot-style__*
* data [List]
    * name : *Is string*
    * value [List]
        * *Is numeric*

{{% /details %}}

Manually validating this long JSON would be error-prone, so I soon started researching the Pydantic package, which I had read is recommended for receiving clean data during FastAPI development.

## Validation with the Pydantic Package

[Pydantic](https://docs.pydantic.dev/latest/) offers many features, but its main selling point is data validation. To avoid making things too complicated, let's consider the following situation:

```python
class User:
    name    : str               # Name
    age     : int               # Age
    spouce  : Optional[User]    # Spouse
    friends : List[User]        # Friends
```

In Python, type hints can help linters, but they don't guarantee actual data type consistency. In the extreme case above, the following code would not raise any error unless you explicitly validate it (e.g., `User(spouce=None).spouce.name`). Some use cases for Pydantic include:

* Type enforcement
    * Forcing specific data to follow type hints
    * Allowing automatic casting if possible (e.g., `'23' -> 23`)
    * Disabling auto-casting or raising `ValidationError` if casting is not possible
    * Handling unspecified entries with error, ignore, or custom conditions
* Complex validation
    * Adding complex conditions beyond type checks easily via arguments
        * For integers: min/max, for strings: length, regex, etc.
    * For overly complex conditions, you can write custom validation functions
* Model validation
    * Validating not just individual entries, but also inter-field relationships (custom functions)
        * For example, `if(self.spouce) assert(self.spouce.name != self.name)` can be set as a function
* Class-JSON conversion support
    * Convert class instances to JSON and vice versa
    * All validation is performed automatically during class instantiation and everywhere else, with no extra function calls

In fact, you can use it anywhere user input sanitization is needed. I regret not learning this sooner, as it would have saved me a lot of trouble in past projects.

## Integrating Logfire and Pydantic

On the Pydantic official site, there's a logging tool called Logfire that's compatible with Pydantic. I found this package interesting, and after looking at code from an old project, it made sense why.

```python
# Old project code (partially modified)
def putLog(name, action=None, data=None):
    if action is None:
        logging.info("{:<29} | {:<8} |",
            get_current_time_ISO(), name
        )
    elif data is None:
        logging.info("{:<29} | {:<8} | action = {:<20}".format(
            get_current_time_ISO(), name, action
        ))
    else:
        logging.info("{:<29} | {:<8} | action = {:<20} | data = {}".format(
            get_current_time_ISO(), name, action, data
        ))

putLog("SystemComponentA", "auto-sync", "[sync-ratio = {:.3f}]".format(
    theory_max
))
```

I used to log like this, but there was too much repetitive code and a high possibility of errors, which was not ideal.

## What I Learned...

The reason I hadn't learned Pydantic until now was simply that I never needed such complex data validation logic before. As always, when business logic gets complicated, there's a limit to ad-hoc solutions, and that's when you have to learn new technologies. And after learning something new, I always wonder, "Why didn't I use this earlier?"

But in the end, you learn when you feel the need. And the fastest place to feel that need is always a project. Whether it's the single responsibility principle, testing, coverage, AWS, Python, Websocket, Agile, or OS, I learned them because I needed them. Inductively, I guess my future self will learn Logfire too, when the need arises. I think it is a great blessing to enjoy learning.

**(Update: 2025.05.20.)** I actually did end up learning it. Unlike my previous ad-hoc Python development, using the Java Spring Boot framework showed me that these issues are already formalized solutions. The Spring equivalent of Pydantic is the Validation framework, and for Logfire, you can use Logback in Java Spring.

I also realized that these issues are ultimately connected to operations. Unlike simple development environments, real operating services make logging, observability, and error handling extremely important. This experience was a big help in understanding Aspect Oriented Programming, which separates these concerns from business logic, when I first learned Spring Boot. ■