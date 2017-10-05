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

> Step 1 Codebase: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step1

For this tutorial, we went with good old
[TodoMVC Single Page App](http://todomvc.com),
adopted and transformed by our team into
[TodoMVC Serverless App](https://github.com/MitocGroup/deep-microservices-todomvc).
Similar approach can be applied to serverless applications described in
[Serverless Stack](https://serverless-stack.com) or
[Serverless Single Page Apps](https://pragprog.com/book/brapps/serverless-single-page-apps),
as well as any other applications from
[Curated List of Awesome Serverless](https://github.com/anaibol/awesome-serverless).

```ssh
# Download locally todomvc serverless application codebase
git clone git@github.com:MitocGroup/deep-microservices-todomvc.git

# Download locally tutorial repository codebase
git clone git@github.com:MitocGroup/tutorial-ci-for-serverless.git

# Copy serverless application into a new branch, part of tutorial repository
cd ./tutorial-ci-for-serverless
git checkout -b tutorial-step1
cp -R ../deep-microservices-todomvc/src ../deep-microservices-todomvc/bin .
git commit -a -m "tutorial step 1"
git push --set-upstream origin tutorial-step1
```

### Step 2: Setup Travis CI

> Step 2 Codebase: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step2

Travis CI takes care of running your tests and deploying your apps. Quoting
official [Getting Started](https://docs.travis-ci.com/user/getting-started/)
guide:

> To start using Travis CI, make sure you have all of the following:
> - GitHub login
> - Project hosted as a repository on GitHub
> - Working code in your project
> - Working build or test script

Here below is our initial `.travis.yml` file:

```yaml
language: node_js
dist: trusty
git:
  depth: 1
cache:
  bundler: true
  directories:
    - node_modules
    - "$(npm root -g)"
    - "$(npm config get prefix)/bin"
branches:
  only:
    - master
node_js:
  - 6
  - 8
script: echo "Hello World!"
```

### Step 3: Setup Unit Testing

> Step 3 Codebase: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step3

Travis CI inspired us with its simplicity and streamlined process. On the other
hand, we were surprised that triggering tests execution is not part of the
service. And we couldn't find any other tool that does it well, so we wrote
one: https://www.npmjs.com/package/recink

Similar to `.travis.yml`, we manage test execution through `.recink.yml`:

```yaml
---
$:
  emit:
    pattern:
      - /^src.es6\/lib\/.+\.js$/i
      - /^test?\/.+\.js$/i
    ignore:
      - /^(.*\/)?bin(\/?$)?/i
      - /^(.*\/)?node-bin(\/?$)?/i
      - /^(.*\/)?node_modules(\/?$)?/i
      - /^(.*\/)?vendor(\/?$)?/i
  npm:
    scripts: []
    dependencies:
      chai: 'latest'
  test:
    mocha:
      options:
        ui: 'bdd'
        reporter: 'spec'
    pattern:
      - /.+\.spec\.js$/i
    ignore: ~

### Add other modules here...
task-create:
  root: src/deep-todomvc/backend/src/task/create
task-delete:
  root: src/deep-todomvc/backend/src/task/delete
task-retrieve:
  root: src/deep-todomvc/backend/src/task/retrieve
task-update:
  root: src/deep-todomvc/backend/src/task/update
```

In order to let Travis CI know that we have some tests to execute, we go back
to `.travis.yml` and change the script that will be executing from `echo "Hello World!"`:

```yaml
before_install:
  - npm install -g recink
script: recink run unit
```

### Step 4: Setup Code Climate

> Step 4 Codebase: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step4

To be updated