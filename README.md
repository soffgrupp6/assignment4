# Report for assignment 4

## Project

Name: williamfiset/Algorithms

URL: https://github.com/williamfiset/Algorithms/

This repository has a variety of algorithms and data structures implemented with Java.

## Onboarding experience

We went with the same project as the previous assignment so there was no new onboarding. In the last assignment our onboarding went well as the only requirements for running tests were gradle and the installation was trivial. Building and running the tests from gradle was only the matter of running `gradle test` in the terminal.

## Effort spent

For each team member, how much time was spent:

| Topic                                 | Anja | Christian | Daniel | Timmy |
|---------------------------------------|-----:|----------:|-------:|------:|
| Preliminary discussions               |   2  |         2 |      2 |     2 |
| Discussions within parts of the group |   0  |         0 |      0 |     0 |
| Reading documentation                 |      |           |        |       |
| Analyzing code/output                 |      |           |        |       |
| Writing documentation                 |      |           |        |       |
| Writing code                          |      |           |        |       |
| Running code                          |      |           |        |       |


## Overview of issue(s) and work done.

Title: Refactor and add tests for Skip List implementation.
URL: https://github.com/williamfiset/Algorithms/issues/74

The issue is about refactoring an implementation of a Skip List and adding of a complete new test suite. The scope of the issue is very local to the file with the Skip List implementation and the test file to be created.

When we started looking at this issue, we intended to restructure and refactor the code, and write some tests for the implementation as specified by the issue. However, after analyzing the code it soon became clear that the code needed not only refactoring, but actually rewriting. Many methods were not only messy but also buggy or simply wrong. Some parts had unresonable implementations of methods. Tests were alse completely missing, probably because the code did not work.

The work needed to resolve this issue was thus much more extensive than we first expected.

## Requirements affected by functionality being refactored

Optional (point 3): trace tests to requirements.

## Code changes

### Patch

(copy your changes or the add git command to show them)

git diff ...

Optional (point 4): the patch is clean.

Optional (point 5): considered for acceptance (passes all automated checks).

## Test results

Overall results with link to a copy or excerpt of the logs (before/afterrefactoring).

## UML class diagram and its description

### Key changes/classes affected

Optional (point 1): Architectural overview.

Optional (point 2): relation to design pattern(s).

## Overall experience

What are your main take-aways from this project? What did you learn?

How did you grow as a team, using the Essence standard to evaluate yourself?

Optional (point 6): How would you put your work in context with best software engineering practice?

Optional (point 7): Is there something special you want to mention here?
