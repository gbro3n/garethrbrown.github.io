---
public: true
layout: post
title: "Fail Early, Fail Fast and why you should throw exceptions"
date: 2018-08-07 00:00 +0000
tags: nlog net-core
---

Debugging is a necessary part of software engineering. All software has bugs, and so debugging goes with the territory. Some are simple to fix. Some are only show up under certain (sometimes unusual) conditions, maybe after an operation has been running for some time or if that operation occurs at a certain time of day. Sometimes applications catch bugs and produce a helpful message or  a stack trace in logs that can help us lead us to where the bug occurred. And sometimes they don't because exceptions are trapped and that information discarded.

It's rare, but most developers have experienced bugs during their career that take them away from feature development for a day or even a week at a time.

Such bugs often present them in a form such as:

- "The application locks up at around 10am every morning and we don't know why."
- "The user clicks the button but nothing happens, and nothing shows up in the console."
- "The customer says they placed the order but we can't see it in the back end."
- "The wait message shows indefinitely"

There is a temptation for programmers to attempt to build robust applications by trapping exceptions and trying to 'handle' them. This maybe the correct action if the error condition is known to be frequent and there is a clear way of working around the error. For example when making calls to an SMTP server that is known to sometimes reject requests, or uses a less than robust network connection, the programmer might want to build in a retry mechanism. Of course it is useful for users or administrators of a system to have to deal with as few failures as possible.

All too often however, programmers catch exceptions without fully 'handling' the error. It might be that a program continues to run without a required value because the exception thrown due to a failed database connection was trapped and the method allowed to return a null value. Such a program might not fall over completely, but it may behave strangely, or not complete subsequent operations in the way that was intended.

The presumption has to be that every line of code in your application does something important. If not, why is it there? Allowing it to work with incorrect or missing data should be expected to be problematic. The resulting types of bugs, where something is 'wrong', rather than the application having visibly failed are usually harder to track down than the equivalent situation where the application has been allowed to fail.

###Fail Fast, Fail Early
The solution to the problem is to 'fail fast, fail early'. Exceptions are called exceptions because they occur in exceptional circumstances. The programmer is not expecting the user to experience them regularly. Clear and visible failure prompts a fix, and makes the cause of the exception easier to find. In the long term, this is what results in robust software.



### However, Don't (Always) Crash the Application
Making failure visible does not mean crashing the application. Presenting the user with technical looking error messages is rarely appropriate. In some types of application, such as software that runs on the desktop or as a service, allowing a crash is likely not acceptable, and for mission critical software applications potentially unsafe.



### Making Failure Visible
So if an application crash is not acceptable, then some sort of error handling is necessary. This should take the form of a global exception handler. Where programs run multiple threads, trap exceptions at the highest possible level, either within the thread or, if your language / framework allows, by assigning an exception handler to the thread.

Ultimately, the operation that the application was attempting should stop, even if the application itself does not fully. The error should then be logged for the attention of a developer.

Where the program has a user interface and the operation was intended to result in visible feedback to the user, a user friendly error view should be presented to the user. This may prompt the user to report the issue. If that is likely, then it might be useful to print the date and time on the screen, and ask the user to take a note of what they were doing at the time when the error occurred.

### Prioritising Errors 
Some errors are expected frequently, such as 404 errors when you run a public facing web application. Content that no longer exists or where bots or users attempt to guess URLs that never existed result in 404s. It might be useful to review these 404s intermittently, but it's unlikely that you want to know about every one. Application errors, and fatal errors that do crash a process or sub process likely require more attention. Some development teams write or copy logs to a centralised location and make failures visible on a UI that everyone can see. Smaller applications might email out errors to system administrators. There are services like Splunk that help to process log data, useful where large volumes of log information are created.



### Final Note

So if you're going to fail, do it fast, visibly, and make sure you collect the information you need to direct you to the source of the problem. While this may result in more visible errors in early life for an application, it will result in more robust, easy maintain software in the future.



Further reading on this topic can be found below.

[https://www.martinfowler.com/ieeeSoftware/failFast.pdf](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)

[https://stackoverflow.com/questions/2807241/what-does-the-expression-fail-early-mean-and-when-would-you-want-to-do-so](https://stackoverflow.com/questions/2807241/what-does-the-expression-fail-early-mean-and-when-would-you-want-to-do-so)