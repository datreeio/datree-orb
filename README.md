# Datree Orb

Easily integrate [datree GitOps policy engine](https://datree.io/ "datree") into your [CircleCi](https://circleci.com/ "circleci") projects. Create custom policies and control version alignment of packages across multiple repositories.

This [orb](https://github.com/CircleCI-Public/config-preview-sdk/blob/master/docs/using-orbs.md "orb") enables to run the custom rule with [CircleCi](https://circleci.com/ "circleci"), on every new build / pull request.

## Use Cases

Developers would like to align and control versions of packages across different projects and teams:
* Version alignment of UI packages - Avoid discrepancy between different versions of the same package of UI components across different projects.
* Version alignment of Internal packages - Avoid breaking changes between different versions of the same internal package across different projects.
* Open source - Create a blacklist of risky versions of specific open source packages.


## Usage

Example config:
```yaml
orbs:
  datree: datree/compare-versions@dev:0.0.6

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
          payload: '{
            "expected": [
              {
                "name":"webpack-node-externals",
                "category":"npm","version":"1.7.2"
              }
            ],
             "actual": [
              {
                "name":"webpack-node-externals",
                "category":"npm",
                "version":"1.6.0"
              }
            ]
          }'
```
`compare-versions` from the `datree` namespace is imported into `datree` which can then be referenced in a step in any job you require.

## Parameters
- ### API_KEY
Enter either your datree api key or use the CircleCI UI to add your token under the ‘DATREE_API_KEY’ env var
Example of providing an `API_KEY` instead of passing a CircleCI context:
```yaml
workflows:
  build_and_test:
    jobs:
      - datree/compare-versions:
          API_KEY: $MY_DATREE_API_KEY
          file_path: ./versions.json
```

- ### payload

A valid json structure:
```json
"expected":[
  {
    "name":"package name (e.g. webpack-node-externals)",
    "category":"package manager (e.g. npm)",
    "version":"required version to verify by datree (e.g. 1.7.2)"
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

- ### file_path
An optional parameter for a json file path instead of the 'payload' parameter
YAML example with `file_path` usage:
```yaml
orbs:
  datree: datree/compare-versions@dev:0.0.6

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
