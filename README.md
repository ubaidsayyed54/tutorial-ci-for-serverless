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

> Step 1 Branch: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step1

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

> Step 2 Branch: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step2

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
node_js:
  - 6
  - 8
script: echo "Hello World!"
```

And there you go, our first successful build:
https://travis-ci.org/MitocGroup/tutorial-ci-for-serverless/builds/283669984

### Step 3: Setup Unit Testing

> Step 3 Branch: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step3

Travis CI inspired us with its simplicity and streamlined process. On the other
hand, we were surprised that triggering tests execution is not part of the
service. And we couldn't find any other tool that does it well, so we wrote
one: https://www.npmjs.com/package/recink

Similar to `.travis.yml`, we manage test execution through `.recink.yml`:

```yaml
---
$:
  npm:
    scripts: []
    dependencies:
      chai: 'latest'
  emit:
    pattern:
      - /^src.es6\/lib\/.+\.js$/i
      - /^test?\/.+\.js$/i
    ignore:
      - /^(.*\/)?bin(\/?$)?/i
      - /^(.*\/)?node-bin(\/?$)?/i
      - /^(.*\/)?node_modules(\/?$)?/i
      - /^(.*\/)?vendor(\/?$)?/i
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

In order to let Travis CI know that we have some tests to execute, we go
back to `.travis.yml` and change the script that will be executing from 
`script: echo "Hello World!"` to this:

```yaml
before_install:
  - npm install -g recink
script: recink run unit
```

### Step 4: Setup Cache

> Step 4 Branch: https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step4

To speed up each Travis CI job, we have built an extra caching layer that
stores npm cache in S3 and reuse it down the road. To enable this feature,
both `.travis.yml` and `.recink.yml` must be updated.

First, in order to work with S3, we need AWS credentials that can't just be
stored in plain text. Travis CI offers native capability to encrypt sensitive
information. Here below is how to do that:

```ssh
# Install Travis CI cli
gem install travis

# Encrypt AWS access key id
travis encrypt -a -x "AWS_ACCESS_KEY_ID=[replace_with_your_key]"

# Encrypt AWS secret access key
travis encrypt -a -x "AWS_SECRET_ACCESS_KEY=[replace_with_your_key]"

# Encrypt AWS region
travis encrypt -a -x "AWS_DEFAULT_REGION=[replace_with_aws_region_where_your_s3_bucket_is]"
```

As a result, you'll have an updated `.travis.yml` with the following several lines:

```yaml
env:
  global:
    - secure: "G8DMndNEZ/E6lFgEqTBHvet1LcR/jr7y5i0Okg5sLZ5guK7lzT3jTI91H7RfGdkOnYAEyjUQ7LLPnGKLIgMtSCvmJbYr7nHQaYWVLkZXg+vmcqRKfLs1x6tclIkLsXlC1q6HYc+wLgJpV/km79PDZnEwL56+MOuntUtgks2hFtVQrMI2M8x9oCLuO+5QVk7KT1UfdnBYe+Iark+8xAuUkEpzLTC4qzrnpTz7ecCQizJkwKlByWn/Rns3nT/xdgASJRVkzg7QWyF5mFiDg5M4g7BuGVafu87eZFZ1YuUPpW18mTqKr45y6m4SXRNnZW5mXFzRpBA4GeTgkL2uYQJJtyRVnijhWM6S7lMZzftJlTeoHHHLGMl8/XyVF6a70feA8da2xvltNRM9HemO9tYRqudowcCl/RiNoutgqUy9vEeZ3HaPW80xybGyUAZi8UfdvA0f/4WevrHKFCJAxYTE5LAzyK41RwTSLRj7x0rW4pIVsn/R329psp1hKkbTiTar+p/sJfKHTman9Qw+BDpHsqpD9ZO/BBpdogM75M0kxiFc6SPG6gAAeLVnolTlYbxihqhKtQOgUCcWCXs62kJNxbH/xow78U3nCGSW5Gjky4BDmt3ekRGZa1w40I66SVqkWfKbysYdp80pU++3ZoeI/w46Jbqowt7xTXLE2UL5/TQ="
    - secure: "sEnXrP26tm/2Wy3EHy6/cMN5zinHiFyuzzKwEzB8rrYRwYpSdpLwWJaqgr9e+NbXJc2CVCSgBfmZkA/ZotTuMYuH77AbjYZKt4XtCC/AmfX9Uqmnw8vu2GRUN0bcqtohML64XjnRu11LcC+uNToja9yfldHY2QbVv6Dvr2IbSg86p1AkRFFcKjfQBoQZBEqyDv32y+Maa3c5qrQMvJBHzc0i9SEis5/crsghLlDK7JTzMDSCpqu6GGxjOyx5BjpgPAQPqP4BcyPKu+DqcJAGA+ThHmCdxM/BIqUbrqoWa72utNqhtXQm1OQYLYVJzgzgwS0GO3I3UIO+kY52E32buwxNBlJJIrQJgW6FcD4ztK/feDaGGIw6sb12HyCBt81hxhsWAuAHZwxXfk9b7EFp0p2edPqzCcJsZ9d/M5BAuHNWV8kNQAifA0hHq9NjP7Ymu3Uq4F6LADk8IU5sUF1cNkt4kWxnqbgZCi7eR7wSxu2ZhrQO7qUOmI+PNAOLUdeONUU3vVcCJIbE6GIvpCiy+xD3HHFc70noItAdFDr0gch6Axv3cDI2Bge1iTXYHqnS4tAbqBcsd1Yqs1WU+Y4PeL/PJt1oRK23u6deJrIVwKxMjUs7ynByVywX8GkH7YRSJmc6Kw8/8ZUq5Fa7Gxte70b6YbyiBHE9NEQkSvr+X54="
    - secure: "MAE2tai6sfIaBAs6rRzjw9m+/1CpknpyXDiNkmH5bvcGsLrQsVDbuxF2S4swDCQcYi8mCFMKzjyPk3i/O03clgKkpqDLrtxmo6DhPZzHB8UKDp27VnemCVMlcYphwKwE57JngzhFZFu9JdttgnpoUfjwW+/pml80g/8ULlUAjyocCOOTOhYaXB44fZOhwT7cB3oVHP8inGMpWvGowyH6skpZUVmkJ5nEoFhKxM/z5kA8CdPBj/7m0JEgOi8knJg1cjQE7TEZRZ94SFMGz/4qZYiTFohqlm/6LjVFFFVupe+OsuluNIKu/r7aFm1wQrIZpOFBEG64tGkYXn0kmMcZ3glUAeqGDDrQ2vEMTfo4gKgOjyKcUloIJ3kuz/EjKiYfCFRYlxKRdIRxc389YaJ+SMf1y7ePBu0+PtL/dBbjTmo5sTylfLfvKesUjXtNJsx3YKI9q6bwHR2p6qGZWIo7WQtr4mH079Nn9vA0I+l2eS2hMhqoEXdVrrFsg8SmdqATsT/SkOAfeBqZrMccG0B1CHFuoSetaVK7OUUcLRBnSULiZSortoHLvLW3KGaGGelRebUMWJvLIl1xnKaT496+IwOGsPtUraqa1fBN4QfBnNGMvsTY1YfMCwALmfXZuGP6imxhcAFx9lKU1yBJPN0HGwlbbZ8mqLJU0YfxDQhvTa4="
```

Second, in order to allow npm cache to be stored in S3, add the following
several lines to `.recink.yml` (the order is not relevant, but for consistency
we decided to put it after unit testing configuration):

```yaml
  cache:
    driver: 's3'
    options:
      - 's3://[replace_with_your_s3_bucket]/[replace_with_your_s3_path]'
      -
        region: 'process.env.AWS_DEFAULT_REGION'
        accessKeyId: 'process.env.AWS_ACCESS_KEY_ID'
        secretAccessKey: 'process.env.AWS_SECRET_ACCESS_KEY'
  preprocess:
    '$.cache.options.1.region': 'eval'
    '$.cache.options.1.accessKeyId': 'eval'
    '$.cache.options.1.secretAccessKey': 'eval'
```

[Click on Continue](https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step5#step-5-setup-code-climate)