# 🎭 Currents - Playwright - AWS CodeBuild

This repository showcases running [Playwright](https://playwright.dev/) tests on AWS Codebuild in parallel while using [Currents](https://currents.dev) as the reporting dashboard.

<p align="center">
  <img width="830" src="https://static.currents.dev/currents-playwright-banner-gh.png" />
</p>

## Documentation

The repo contains a few Playwright tests with one test that always fails (intentionally). The example [buildspec.yml](https://github.com/currents-dev/aws-codebuild-example/blob/main/buildspec.yml) defines a configuration for running cypress tests in parallel mode using 3 workers in [matrix mode](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build.html#batch_build_matrix). The example is designed to be executed as a [batch build](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build.html).

To reproduce the setup:

- Create a new Build Project in AWS CodeBuild - set the resource class and AWS-specific configuration according to your needs.

- Create an organization, get your **Record Key** and **Project Id** at https://app.currents.dev
- Save **Record Key** as `CURRENTS_RECORD_KEY`[Environment variable](https://docs.aws.amazon.com/codebuild/latest/userguide/change-project-console.html#change-project-console-environment). Treat this variable as a secret - i.e. store it in a secure storage.
- Update the command in `buildspec.yml` with the **Project Id**

### AWS CodeBuild Configruation Note

- The example uses [CODEBUILD_INITIATOR](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html) as a [CI Build ID](https://currents.dev/readme/guides/cypress-ci-build-id). When testing interactively, the CODEBUILD_INITIATOR will be set to the username of the build initiator. When running a batched build, the variable will have the batch build ID.
