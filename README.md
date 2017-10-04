# Tutorial: Continuous Integration for Serverless Applications

Continuous Integration is the practice of merging all working copies of
developer code into one shared mainline several times a day. Best practices
include automation of builds and deployments, with fast and self-testing
builds, as well as production-like testing environments. With serverless, the
continuous integration pipeline evolves from a one-lane, one-way street into a
multiple-lane, two-way highway.

In this tutorial, we take a simple serverless application and walk you through
the steps to set up unit testing, end-to-end testing, code coverage, code
analysis, code security, code performance, and peer code review. You can learn
how to use AWS serverless components in combination with GitHub, Travis-CI,
CodeClimate, Snyk and other serverless-friendly services.

## Getting Started

### Step 1: Identify Your Serverless Application

For this tutorial, we went with good old [TodoMVC example](http://todomvc.com).
Similar approach can be applied to serverless applications described in
[Serverless Stack](https://serverless-stack.com) or
[Serverless Single Page Apps](https://pragprog.com/book/brapps/serverless-single-page-apps),
as well as any other resource described in
[Curated List of Awesome Serverless Applications](https://github.com/anaibol/awesome-serverless).

Since we have already adopted TodoMVC using Serverless Architecture, this
tutorial will reuse that open sourced code:

```ssh
git clone git@github.com:MitocGroup/tutorial-ci-for-serverless.git
git clone git@github.com:MitocGroup/deep-microservices-todomvc.git
cp -R deep-microservices-todomvc/src deep-microservices-todomvc/bin tutorial-ci-for-serverless/
```

[Click to Continue](https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step2)