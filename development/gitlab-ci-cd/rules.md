# GitLab CI/CD rules

> GitLab introduced the `rules` keyword in version 12.3. This is the preferred way of specifying when jobs run since `only` and `except` are not being actively developed ([source](https://docs.gitlab.com/ee/ci/yaml/#only--except)).

## Table of Contents

- [Base concept](#base-concept)
- [Examples](#examples)
  - [Only on tags](#only-on-tags)
  - [Only on default branch](#only-on-default-branch)

## Base concept

Rules are defined directly on the job level, like this:
```yml
build-job:
  stage: build
  rules:
    - ...
```

It seems that using the predefined variables is the way to go in terms of detecting the current environment. You can find all predefined variables [here](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html).

## Examples

### Only on tags

> Since the variable `$CI_COMMIT_TAG` is only available in tag pipelines, you can make use of this.

```yml
rules:
  - if: $CI_COMMIT_TAG
```

### Only on default branch

> Check if the branch name of the commit is equal to the default branch name.

```yml
rules:
  - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```
