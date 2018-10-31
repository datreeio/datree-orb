# Datree Orb

Easily integrate [datree GitOps policy engine](https://datree.io/ "datree") into your [CircleCi](https://circleci.com/ "circleci") projects. Create custom policies and control version alignment of packages across multiple repositories.

This [orb](https://github.com/CircleCI-Public/config-preview-sdk/blob/master/docs/using-orbs.md "orb") enables to run the custom rule with [CircleCi](https://circleci.com/ "circleci"), on every new build / pull request.

## Use Cases

Developers would like to align and control versions of packages across different projects and teams:
* UI packages versioning alignment - Avoid discrepancy between different versions of the same package UI components across different projects.
* Internal packages versioning alignment - Avoid breaking changes between different versions of the same internal package across different projects.
* Open source - Create a blacklist of risky versions of specific open source packages.


## Usage

Example config:
```yaml
orbs:
  datree: datree/compare-versions@dev:0.0.1

workflows:
  build_and_test:
    jobs:
      - datree/run-version-compare

```
`version-alignment-rule` from the `datree` namespace is imported into `datree` which can then be referenced in a step in any job you require.

## Commands
- ### API_KEY
Enter either your datree api key or use the CircleCI UI to add your token under the ‘DATREE_API_KEY’ env var

- ### payload

A valid json stracture*:
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
\*Path to a json file is also supported
