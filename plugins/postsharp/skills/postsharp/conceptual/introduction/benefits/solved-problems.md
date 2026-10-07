---
uid: solved-problems
title: "Problems PostSharp Solves"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp addresses problems in conventional programming such as large codebases, reinventing solutions, high defect rates, multithreading issues, lack of robustness, code maintenance difficulty, strong coupling, slow onboarding of new team members, and long feedback loops."
---
# Problems PostSharp Solves
Conventional programming languages miss a concept of pattern, therefore patterns are hand-coded and result in boilerplate code.


## High development effort

* **Large codebases**. Some application features require a large amount of repetitive code (boilerplate) when implemented with existing mainstream compiler technologies. 

* **Reinventing the wheel**. Solutions to problems like INotifyPropertyChanged are always being reinvented because there are no reusable options within conventional programming languages. 


## Poor quality software

* **High number of defects**. Every line of code has a possibility of defect, but code that stems from copy-paste programming is more likely than other code to be buggy because subtle differences are often overlooked. 

* **Multithreading issues**. Object-oriented programming does not deliver much value when it comes to developing multithreaded applications since it addresses issues at a low level of abstraction with locks, events or interlocked accesses that can easily result in deadlocks or random data races. 

* **Lack of robustness**. Enterprise-grade features such as exception handling or caching are often deliberately omitted because of the high amount of source code they imply, unintentionally forbidden in some parts of the applications, or left untested and unreliable. 


## Difficulty adding or modifying functionality after release 1.0

* **Unreadable code that’s difficult to maintain**. Business code is often littered with low-level non-functional requirements and is more difficult to understand and maintain, especially when the initial developer has left. 

* **Strong coupling**. Poor problem decomposition results in duplicate code and strong coupling making it very difficult to change the implementation of features like logging, exception handling or INotifyPropertyChanged because it is often scattered among thousands of files. 


## Slow ramp-up of new team members

* **Too much knowledge required**. When new team members come to work on a specific feature, they often must first learn about caching, threading and other highly technical issues before being able to contribute to the business value: an example of bad division of labor. 

* **Long feedback loops**. Even with small development teams, common patterns like diagnostics, logging, threading, and INotifyPropertyChanged can be handled differently by each developer. Architects need to make sure new team members understand and follow the internal design standards, and they have to spend more time on manual code reviews—delaying progress while new team members wait to get feedback from code review. 


