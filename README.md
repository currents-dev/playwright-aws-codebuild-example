# 🎭 Currents - Playwright - AWS CodeBuild

This repository showcases running [Playwright](https://playwright.dev/) tests on AWS CodeBuild in parallel while using [Currents](https://currents.dev) as the reporting dashboard.

<p align="center">
  <img width="830" src="https://static.currents.dev/currents-playwright-banner-gh.png" />
</p>

## Documentation

The repo contains a few Playwright tests, one test always fails (intentionally) for demonstration.

The example [buildspec.yml](https://github.com/currents-dev/aws-codebuild-example/blob/main/buildspec.yml) defines a configuration for running Playwright tests in parallel mode using 3 workers in [matrix mode](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build.html#batch_build_matrix). The example is designed to be executed as a [batch build](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build.html).

## Results

The results are being reported to Currents for more efficient troubleshooting, and monitoring test suite flakiness and performance.

Currents will collect the following information:

- commit details
- console output
- screenshots
- videos
- trace files
- timing
- outcomes
- flaky tests
- error details
- tags for more convenient management of the tests

The example repository uses an AWS CodeBuild project that runs Playwright tests in parallel on 3 containers using [Playwright Sharding](https://playwright.dev/docs/test-parallel#shard-tests-between-multiple-machines).

Here's an example of a build that has 1 batch job (1) triggering 3 parallel build jobs (2):

The corresponding run in Currents dashboard:

## AWS Setup

Please refer to the [sample output of AWS CodeBuild project configuration](./aws-project-config-output.json) used for this demo. The output was generated using this command: `aws codebuild batch-get-projects --names playwright-currents-example`. Alternatively, follow the instructions below.

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

## Playwright Parallelization / Sharding

The `buildspec.yml` file uses [matrix mode](https://docs.aws.amazon.com/codebuild/latest/userguide/batch-build) to start 3 containers for running the test in parallel. Each container will have the environment variable `WORKER` set to `1,2,3` correspondingly - we can use it to configure [Playwright Sharding](https://playwright.dev/docs/test-parallel#shard-tests-between-multiple-machines)

```yml
  build-matrix:
    dynamic:
      buildspec:
        - buildspec.yml
      env:
        variables:
          # Create 3 containers, each container will have the environment variable `WORKER` set to `1,2,3`
          WORKER:
            - 1
            - 2
            - 3

        # and later, note the use of $WORKER env variable
      - npx pwc --project-id bnsqNa --key $CURRENTS_RECORD_KEY --ci-build-id $CODEBUILD_INITIATOR --shard $WORKER/3
```

## CI Build ID for AWS CodeBuild

The example uses [CODEBUILD_INITIATOR](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html) as a [CI Build ID](https://currents.dev/readme/guides/cypress-ci-build-id). If you trigger the build manually, the `CODEBUILD_INITIATOR` will be set to the username of the build initiator and you can get warnings after recording multiple results for the same CI build ID.

When a build is triggered by a push / PR (and a batched build is created), the variable will have a unique ID associated that won' trigger conflict warnings.

## Using Currents Reporter

The example uses `pwc` CLI command to run the tests. You can use `npx playwright test` command and configure `@currents/playwright` as a reporter. Please refer to the [documentation](https://currents.dev/readme/integration-with-playwright/currents-playwright#currents-playwright-reporter).

## Running Locally

Based on: https://docs.aws.amazon.com/codebuild/latest/userguide/use-codebuild-agent.html

```sh
curl -O  https://raw.githubusercontent.com/aws/aws-codebuild-docker-images/master/local_builds/codebuild_build.sh
chmod +x codebuild_build.sh

# For MacBook M1
./codebuild_build.sh -i public.ecr.aws/codebuild/amazonlinux2-aarch64-standard:3.0 -l public.ecr.aws/codebuild/local-builds:aarch64

# For Intel
./codebuild_build.sh -i public.ecr.aws/codebuild/amazonlinux2-standard:3.0
```
