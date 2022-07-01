# Podinfo Example CI/CD project
This repo provides a basic example of using GitHub Actions with Flux for CI/CD.  The layout and pipeline flow is designed to mimic a potential real world deployment pattern.  It uses a single repo with a branch strategy to separate application code from environment deployment.

## Repo layout
The `main` branch is designed to be the home of all deployment work.  It contains the Dockerfile needed to make app changes as well as an app directory which holds the base config necessary to deploy the app to a cluster.

The `release` branch contains all the env specific configs necessary to deploy the app.  This includes specific image versions, which namespace to use, etc.  This is the branch and directories that Flux will be configured to use.

## Pipelines
In the main branch there is a single workflow defined.  This workflow will automatically build new Docker images and tag them based on the branch, tag, or pull request (depending on what triggered the action).  This image will only be pushed for branch or tag events.  PR events are only used to test the build process and make sure it succeeds correctly.  All pushes to main or newly created tags are automatically built and deployed to the dev environment.

In the release branch that is another set of workflows that handle the semi-automated release of the app to the staging and prod environments.  These workflows detect changes to the previous environment and automatically create branches, commit changes, and create PR's for the next target environment.  For example, changes to the dev environment will trigger a PR to be opened for the stagging environment and changes to the stagging environment with open a PR for the prod environment.

> Using a single repo for app and release config was used for demo simplicity.  The patterns used here could easily be migrated to a multi-repo or multi-directory setup.

## Pain points
### Flux setup
The Flux setup for release deployments was simple and straight forward.  Take a look at the [flux.md](flux.md) file for setup details.

### Pipeline setup
The pipeline setup was much more difficult.  The biggest challenge was adding conditionals to restrict when stages should run.  You would think this would be pretty trivial, but it wasn't.  Here is a list of the key obstacles and the steps taken to overcome them (I am excluding my own hurdles of learning GitHub actions as that was more of a me issue and not a pipeline issue)

- Need to update image tag automatically on commit to `main` or new tags
  - I've used various options before including creating deployment templates and using `envsubst` to create deployment yaml(s).  This time I went with a pure kustomization approach and used an [env configMapGenerator](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/configmapgenerator/#configmap-from-env-file) along with [replacements](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/replacements) to overwrite the image tag.  With this approach I was able to dynamically create an `image.env` file in my pipeline and pass that file to release configs.
- Need to trigger pipelines on more than just image updates
  - One issue with using kustomization bases is that changes to the base file(s) do not show up on a PR.  To fix this I updated the build pipeline to build the kustomize base directory into a single `app.yaml` file that would be released to the environments instead.  I create a new [kustomize stage](https://github.com/rparmer/podinfo/blob/main/.github/workflows/build-and-deploy-dev.yaml#L76) to generate the file.  With this approach the PR would also show changes to the deployment, service, sa, etc.
- Changes to the `main` image where not deploying to the dev environment
  - Since the image tag did not change, the deployment did not update on the cluster.  To overcome this I added a `Get build date` stage to the workflow and added a `image.timestamp` label to the dev deployment spec template.  That date was then added to the `image.env` file added to the dev kustomization.yaml file [here](https://github.com/rparmer/podinfo/blob/release/dev/kustomization.yaml#L24).  I also update the dev image pull policy to be Always [here](https://github.com/rparmer/podinfo/blob/release/dev/kustomization.yaml#L36).
  > These updates to the kustomiztion file were only required for dev.
- Commits from the pipeline were not triggering new workflows to start
  - Turns out GitHub actions have a safeguard in place to prevent pipelineception.  They do this by not allowing the default workflow token to trigger new workflows.  Once I knew the issue it was easy to fix by using a custom personal access token instead.  The token was set an actions secret and then referenced [here](https://github.com/rparmer/podinfo/blob/main/.github/workflows/build-and-deploy-dev.yaml#L103).
  > This was only needed on the `release-dev` stage.  All other stages as well as the `release` workflows do not need the custom token.
- Release pipelines would run on updates to the `main` image
  - Multiple ways to clean this up, but since I have the release pipelines configured to trigger on changes to the prior environment I added a `Release` stage like [this](https://github.com/rparmer/podinfo/blob/release/.github/workflows/release-staging.yaml#L24) that was used to indicate if the commit and PR stages should run.
  > Alternative would be to update the workflow `on push` event to include tags.  But that would create PR's for both staging and prod at the same time.  I opted against this since I wanted a progressive flow from dev -> staging -> prod
