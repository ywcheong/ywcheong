---
title: 'sys.setrecursionlimit(10**6) Is NOT a Silver Bullet'
date: '2024-10-31T20:19:49+09:00'
---

## To Begin with the Conclusion...

When solving algorithm problems in Python, if your logic requires exceeding the default recursion limit of 1000, it's better to refactor your code to use iterative functions or consider switching to C or C++. Otherwise, **there is a risk of encountering fatal bugs on Windows that cannot be detected**.

## Where the Problem Started

This all began while I was solving [Baekjoon 11049 (Matrix Chain Multiplication)](https://www.acmicpc.net/problem/11049). This problem is a classic dynamic programming question where memoization is used, and recursion makes the structure clean and intuitive.

Although C or C++ is generally recommended for algorithm problems, I often use Python for various projects and felt comfortable using it here. However, due to the tight time limits set for the problem, I encountered several time limit errors. (For reference, after converting the code to C via ChatGPT, it ran in under 10ms.) To analyze which part of the program was slow, I created a large test case and fed it as input.

The problem was about finding the most efficient way to multiply n matrices, and n was less than 500. I became curious about what would happen if n was larger, so I generated a test case with n = 5000.

![alt text](/python-recursion-bug-1.png "matrix_size = 5000.")

Surprisingly, the program terminated immediately with no output whatsoever.

![alt text](/python-recursion-bug-2.png "Reenactment of the situation. No output from the program. Although not shown, the program had ended.")

There was no RecursionError, which is a common error when doing PS (problem solving) in Python. There were no error messages or anything unusual. Most importantly, if n was reasonably small (e.g., n = 500), the program ran just fine.

I searched many times for a solution, but found very little documentation on the root cause, so I decided to summarize it myself.

## How to Reproduce the Problem

Actually, this issue can be reproduced with a very simple Python script.

```python
import sys
sys.setrecursionlimit(10000010)

def recursion(n):
    if n == 0:
        return 0
    return n + recursion(n-1)

print("Program Start")
print(recursion(10000000))
print("Program End")
```

If you run this code in a **Windows terminal** as `python3 script.py`, you'll see "Program Start" and then the command will exit with no further output. If you run it in the Python REPL, the REPL itself will crash.

This has already been reported in [Python Bugtracker Issue#45645](https://bugs.python.org/issue45645). To summarize: When a recursive function gets too deep on Windows, a Stack Overflow error occurs. On Windows, there's no way to properly catch this error, so Python just dies. When a program exceeds the stack size on Windows, the process is terminated immediately, and the bug was closed as "Won't Fix." Technically, it's possible to create a [Guard Page](https://learn.microsoft.com/ko-kr/windows/win32/memory/creating-guard-pages) for protection, but considering the overhead and the fact that most Python programs don't use such deep recursion, it hasn't been implemented.

### This Bug Does Not Occur on Linux

![alt text](/python-recursion-bug-3.png "This code, which doesn't work on Windows, runs in under 0.01 seconds on Linux. Screenshot from Amazon EC2 Linux.")

On Linux, this bug does not occur. Even with `setrecursionlimit` set to an extremely high value (e.g., 2.1 billion), the code runs fine. This is because Linux and Windows handle process stack size limits differently.

According to the [Python module - resource documentation](https://docs.python.org/ko/3/library/resource.html), Linux provides the `resource` module to directly control stack size, but Windows does not. This is because the `resource` module uses the `setrlimit` system call in Linux, but Windows does not allow dynamic stack size changes at runtime. On Windows, if you want to use more stack space, you have to create a new thread with a larger stack, or use other indirect methods. Also, the default stack size on Windows is generally smaller than on Linux.

{{< callout type="warning" >}}
  The default stack size can change easily depending on compiler options or commands like `ulimit -s stack_size` on Linux. However, the fact that Windows cannot dynamically change stack size at runtime remains true, so the situation doesn't change much.
{{< /callout >}}

In summary, Linux (unlike Windows):

* Can dynamically expand the stack at runtime
* (Usually, but not always) Has a larger default stack size

These factors combine to create the observed differences.

## So, What Should You Do?

To fundamentally solve the problem, it's best not to use recursion. But in PS (problem solving), recursion can be much more intuitive.

In most cases, C or C++ is used for PS, so this problem is less common. C and C++ support tail recursion optimization, but Python does not, leading to significant differences in resource usage. The code that caused the problem above runs in less than 10ms when converted to C.

So, the most appropriate approach is:

* If using C or C++:
    * Use recursion carefully, leveraging tail recursion optimization.
* If using Python or other languages:
    * Use recursion only for small recursion depths (< 1,000).
    * Even then, be aware of the overhead, and avoid recursion when possible.
    * Be very careful about arbitrarily increasing the recursion limit with `sys.setrecursionlimit`.

**(2025.05.20. Update)** In fact, recursion carries risks even aside from performance. Industry standards for mission-critical systems, like [ISO 26262](https://www.iso.org/standard/68383.html), often explicitly prohibit recursion because it complicates static code analysis and makes recursion depth difficult to control. Of course, considering all these factors when solving coding test problems may be overengineering, but it's important to understand the costs of recursion.