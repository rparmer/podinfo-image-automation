# Podinfo Example CI/CD project
This repo provides a basic example of using GitHub Actions with Flux for CI/CD.  The layout and pipeline flow is designed to mimic a potential real world deployment pattern.  It uses a single repo with a branch strategy to separate application code from environment deployment.

## Repo layout
The `main` branch is designed to be the home of all deployment work.  It contains the Dockerfile needed to make app changes as well as an app directory which holds the base config necessary to deploy the app to a cluster.

The `release` branch contains all the env specific configs necessary to deploy the app.  This includes specific image versions, which namespace to use, etc.  This is the branch and directories that Flux will be configured to use.

## Current issues
- With this setup the dev environment will always have the latest version of `main` deployed with staging and prod having the latest `tag`.  For dev and staging this is fine, but for prod this is not desired as you would probably want to verify staging first before deploying to prod.  Prod can be configured to push changes to a new branch, but currently no PR would be auto generated.

- What can be done to auto generate PR's?
