# Teamcity + Octopus feature streams

## Octopus project

Check the octopus project has all the Channels setup.

http://octopus.abcam.com/app#/Spaces-1/projects/net-skeleton/channels

## Git repository

Checkout `master`

```sh
git checkout master && git pull && git submodule update
```

Create one branch per stream

```sh
git branch BAU && git branch stream/F1 && git branch stream/F2 && git branch stream/P1 && git branch stream/P2
```

Push branches to remote

```sh
git push -u origin stream/BAU stream/F1 stream/F2 stream/P1 stream/P2
```

## Teamcity 

### Variables

At the project level override:

- `Build-Number-MajorMinorPatch` in the format of `*.*.*` (`*` are digits only)
- `Channel-Name` with value `Branch-Name`  ( when all project are migrated it will be done in the root level )

### *Build Configurations* - Builds

- General settings
  - set `Build number format` to `%Build-Number-Format%`
  - save
- Build Step: Command Line
  - inside the build step update the command parameter to `buildOnly -buildNumber %Build-Number-dotNetAssemblyVersion%`
- Repeat for each build steps in the project if there are more than 1

### *Build Configurations* - Create OctopusRelease

- Triggers > Finish Build Trigger
  - set `Branch filter` to
    ```
    +:<default>
    +:stream/*
    ```

### *Build Configurations* - Deploy CI

- Change Deploy CI step:
  - set `Build configuration type` to Deployment
  - set `Limit the number of simultaneously running builds (0 — unlimited)` to `0`
  - save
- Build Step: OctopusDeploy: Deploy release
  - check `Environment(s):` is `%Channel-Name%-CI`
- Triggers > Finish Build Trigger
  - clear up `Branch filter` text area  
  - save
- Dependencies
  - check conditions
  
### *Build Configurations* - Integration / Smoke / Regression 

- Build Step: > NUnit
  - setup `Run tests from` to use one of `%Channel-Name%-CI` or `%Channel-Name%-REG` or `%Channel-Name%-PERF` instead of hardcoded environments
- Triggers > Finish Build Trigger
  - clear up `Branch filter` text area  
  - save

### *Build Configurations* - Promote XX

Convert Promote XX to Deploy by (if needed):

- General settings
  - rename the step
  - click *Regenerate ID*
  - change the `Build configuration type`
- Build Step: OctopusDeploy: Promote release
  - change `Runner type` to `OctopusDeploy: Deploy release`
  - fill all the mandatory fields (`API key` is in 1password)
  - save
- [ NOT DO THIS FOR PERF / PP / PROD / DR ] Triggers > Finish Build Trigger
  - clear up `Branch filter` text area
  - save

### *Edit VCS Root*

- General settings
  - set `Default branch` to BAU
  - set `Branch specification` to
   ```
   +:refs/heads/stream/(*)
   +:refs/heads/(BAU)
   ```