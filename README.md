# Datree Orb

Easily integrate [Datree GitOps policy engine](https://datree.io/ "datree") into your [CircleCi](https://circleci.com/ "circleci") projects. Create custom policies and control version alignment of packages across multiple repositories.

This [orb](https://circleci.com/orbs/) enables to run the custom rule with [CircleCi](https://circleci.com/ "circleci"), on every new build / pull request.

## Use Cases

### Connect Branch Name To Issue Tracker (datree/branch-name-convention)
Ease your developer experience by validating that pushing code to your VCS is automatically tracked in your issue tracker.
* :pushpin: Create a logical link between planning (issue tracker such as Jira) and the subsequent code change (GitHub)
* :closed_lock_with_key: Recommended requirement for SOC 2 compliance

### Connect Pull Request Title To Issue Tracker (datree/pull-request-title-convention)
Ease your developer experience by validating that pushing code to your VCS is automatically tracked in your issue tracker.
* :pushpin: Create a logical link between planning (issue tracker such as Jira) and the subsequent code change (GitHub)
* :closed_lock_with_key: Recommended requirement for SOC 2 compliance

### Version Alignment (datree/version-alignment)
Developers would like to align and control versions of packages across different projects and teams:
* :eyeglasses: Version alignment of UI packages - Avoid discrepancy between different versions of the same package of UI components across different projects.
* :lock: Version alignment of Internal packages - Avoid breaking changes between different versions of the same internal package across different projects.
* :no_entry_sign: Open source - Create a blacklist of risky versions of specific open source packages.


## Usage

### Connect Branch Name To Issue Tracker
Example config:
```yaml
orbs:
  datree: datree/policy@latest # we advise to lock the version before pushing to production

description: A circle-ci job that uses Datree's branch name alignment orb

jobs:
  my_job:
    docker:
      - image: circleci/node:10
    steps:
      - datree/branch-name-convention:
          issue_tracker: jira

version: 2.1
workflows:
  main:
    jobs:
      - my_job
```

### Connect Pull Request Title To Issue Tracker
Example config:
Using a token provided as an environment variable
```yaml
orbs:
  datree: datree/policy@latest # we advise to lock the version before pushing to production

description: A circle-ci job that uses Datree's pull request title alignment orb

jobs:
  my_job:
    docker:
      - image: circleci/node:10
    steps:
      - datree/pull-request-title-convention:
          issue_tracker: jira

version: 2.1
workflows:
  main:
    jobs:
      - my_job:
          context: github-token-context # required in order to use the environment variable
```
Using a token as a hard coded string (not recommended - use only for POC purposes).
```yaml
orbs:
  datree: datree/policy@latest # we advise to lock the version before pushing to production

description: A circle-ci job that uses Datree's pull request title alignment orb

jobs:
  my_job:
    docker:
      - image: circleci/node:10
    steps:
      - datree/pull-request-title-convention:
          issue_tracker: jira
          token: '<ADD_GITHUB_TOKEN_HERE>'

version: 2.1
workflows:
  main:
    jobs:
      - my_job
```

### Version Alignment
Example config:
```yaml
orbs:
  datree: datree/policy@latest # we advise to lock the version before pushing to production

description: A circle-ci job that uses Datree's version alignment orb

jobs:
  extract-versions:
    docker:
      - image: circleci/circleci-cli:0.1.2709
    steps:
      - run: echo "running version extraction script"
      # this is a mock demo - running a script to extract the package versions that were installed during the build
  my_job:
    docker:
      - image: circleci/node:10
    steps:
      - datree/version-alignment

version: 2.1
workflows:
  main:
    jobs:
      - extract-versions
      - my_job:
          requires:
            - extract-versions
          context: datree-api-context # required in order to use the environment variable
          payload: '{
            "expected": 
              [{"name":"webpack-node-externals","category":"npm","version":"1.7.2"}],
            "actual":
              [{"name":"webpack-node-externals","category":"npm","version":"1.7.2"}]
          }' # In a real life scenario this is passed from the extract-versions job.
```

## Parameters

### Connect Branch Name To Issue Tracker
- #### issue_tracker
Name of issue tracker. Currently supports Jira (key: `jira`) and Pivotal (key: `pivotal`). Default value is `jira`.

### Connect Pull Request Title To Issue Tracker
- #### issue_tracker
Name of issue tracker. Currently supports Jira (key: `jira`). Default value is `jira`.
- #### token
Enter either a Github token or use the CircleCI UI to add your token under the ‘GITHUB_TOKEN’ environment variable.
The required scopes are `repo`, for a full 'how-to' guide on generating a Github Token [see our documentation](https://docs.datree.io/docs/generate-github-token).

### Version Alignment
- #### API_KEY
Enter either your Datree api key or use the CircleCI UI to add your token under the ‘DATREE_API_KEY’ env var
Example of providing an `API_KEY` instead of passing a CircleCI context:
```yaml
workflows:
  build_and_test:
    jobs:
      - datree/compare-versions:
          API_KEY: $MY_DATREE_API_KEY
          file_path: ./versions.json
```

- #### payload

A valid json structure:
```json
"expected":[
  {
    "name":"package name (e.g. webpack-node-externals)",
    "category":"package manager (e.g. npm)",
    "version":"required version to verify by Datree (e.g. 1.7.2)"
  }
],
"actual":[
  {
    "name":"package name",
    "category":"package manager",
    "version":"actual version installed in the build process (e.g. 1.6.0)" 
  },
]
```

- #### file_path
An optional parameter for a json file path instead of the 'payload' parameter
YAML example with `file_path` usage:
```yaml
orbs:
  datree: datree/policy@latest # we advise to lock the version before pushing to production

jobs:
  extract-versions:
    docker:
      - image: circleci/circleci-cli:0.1.2709
    steps:
      - run: echo "running version extraction script"
      # this is a mock demo - running a script to extract the package versions that were installed during the build

workflows:
  build_and_test:
    jobs:
      - extract-versions
      - datree/compare-versions:
          requires: 
            - extract-versions
          context: datree-api-key
          file_path: ./versions.json
```
