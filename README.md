# 🎭 Currents - Playwright - AWS CodeBuild

This repository showcases running [Playwright](https://playwright.dev/) tests on AWS CodeBuild in parallel while using [Currents](https://currents.dev) as the reporting dashboard.

<p align="center">
  <img width="830" src="https://static.currents.dev/currents-playwright-banner-gh.png" />
</p>

## Documentation

The repo contains a few Playwright tests, one test always fails (intentionally) for the purpose of demonstration.

The example [buildspec.yml](https://github.com/currents-dev/aws-codebuild-example/blob/main/buildspec.yml) defines a configuration for running Playwright tests in parallel mode using 3 workers in [matrix mode](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build.html#batch_build_matrix). The example is designed to be executed as a [batch build](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build.html).

## Setup

### Obtain Currents Credentials

- Create an organization, get your **Record Key** and **Project ID** at https://app.currents.dev.
- Update the command in `buildspec.yml` file with the **Project ID**: e.g. `npx pwc --project-id <your-project-id> ...`

### Configure AWS CodeBuild Build Project

#### Configure `CURRENTS_RECORD_KEY`

Save the **Record Key** as `CURRENTS_RECORD_KEY` [Environment variable](https://docs.aws.amazon.com/codebuild/latest/userguide/change-project-console.html#change-project-console-environment). It is strongly recommended to use your **Record Key** in a secure secrets storage. Please refer to the [detailed guide](https://www.learnaws.org/2022/11/18/aws-codebuild-secrets-manager/), here is an overview of the steps:

- Create a new entry in AWS Secrets Manager with the **Record Key**. Please note that the generated secret is a JSON document, you should note the `json_key` of the actual record key value and use it later.
- Get the secret ARN
- Update the Build Project environment variables as follows:

  - Variable name: `CURRENTS_RECORD_KEY`
  - Variable value: the ARN of previously created secret + json_key, for example: `<secret-arn>:<json-key>`

- Update the IAM execution role to allow reading of previously created secret

#### Configure Source Batch Mode

- Set the **Project Setting > Edit Source**
- Configure the repository details and the events that should trigger new builds
- Configure **Primary source webhook events > Build Type** to **Batch build** to start 3 parallel workers in [matrix mode](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build).

- The example uses [CODEBUILD_INITIATOR](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html) as a [CI Build ID](https://currents.dev/readme/guides/cypress-ci-build-id). When testing interactively, the CODEBUILD_INITIATOR will be set to the username of the build initiator. When running a batched build, the variable will have the batch build ID.

- Create a new Build Project in AWS CodeBuild - set the resource class and AWS-specific configuration according to your needs.

The example uses `pwc` CLI command to run the tests, you can use `npx playwright test` command and configure `@currents/playwright` as a reporter. Please refer to [documentation](https://currents.dev/readme/integration-with-playwright/currents-playwright#currents-playwright-reporter).

## Running Locally

Based on: https://docs.aws.amazon.com/codebuild/latest/userguide/use-codebuild-agent.html

```sh

curl -O  https://raw.githubusercontent.com/aws/aws-codebuild-docker-images/master/local_builds/codebuild_build.sh
chmod +x codebuild_build.sh

# For MacBook M1
./codebuild_build.sh -i public.ecr.aws/codebuild/amazonlinux2-aarch64-standard:3.0 -a ./aws-cb  -l public.ecr.aws/codebuild/local-builds:aarch64
```
