# GovWifi Concourse Deploy Pipeline

Concourse pipeline for managing deployments of all GovWifi services.

## GovWifi Concourse Runner

All tasks will be run using the [GovWifi Concourse Runner][govwifi-runner].

This gives the task access to `docker`, `docker-compose`, `python2`, and `aws-cli`.

## AWS Helpers

for when running the `deploy` task, access is given to some functions that wrap `aws-cli`.

To access, you can source it from the `deploy-tools` folder:

```sh
source deploy-tools/aws-helpers.sh
```

This gives you access to some useful functions:

### `stage_name`

Positional Arguments: none

echo's the stage name used within the GovWifi AWS account.
Utilises the `$STAGE` environment variable.

This amounts to translating `production` into `wifi`.

### `run_task_with_command`

Positional Arguments:

- `cluster_name`: The name of the ECS cluster
- `service_name`: The name of the ECS service within the cluster
- `task_definition`: the family name of the task definition
- `docker_service_name`: The name of the docker service within the task definition to run under
- `command`: The command to run inside the container, as a string, e.g. 'echo "hello, world"'

Runs a task with an explicit command, useful for running one-shot tasks, like database migrations.

### `ecs_deploy`

Positional Arguments:

- `cluster_name`: The name of the ECS cluster
- `service_name`: The name of the ECS service within the cluster

Updates and deploys an existing ECS service.

### `ecs_deploy_region`

Positional Arguments:

- `cluster_name`: The name of the ECS cluster
- `service_name`: The name of the ECS service within the cluster
- `region`: The AWS region to target, e.g. 'eu-west-1' or 'eu-west-2'

Updates and deploys an existing ECS service in a specific region.

## Adding a service

There are some requirements a service must fulfil before adding it to this pipeline:

- Be deployable to a Docker Repository
- should Implement all tasks documented below.

*Note*: while each service can be defined separately, it's best to stick with the same
structure for all services, so that when the time comes we can move to templates.

### Tasks to implement for testing

These tasks should match that of the PR pipeline.

They are mostly self contained, and so we only require that the files exist.

For each service, we can load additional registry-images for caching purposes.

#### pre-build
Location: `ci/tasks/pre-build.yml`

A task to pre-build the service image.

It can define dependent registry-images to help speed up build times.
These must be manually added in the pipeline, and updated against the service docker files.

As this task is consumed by the `test` and `lint` tasks, we do not define an interface
as it is self contained.

That is to say, we care that the task exists, we don't care what it does.

An example can be found [here][example-pre-build].

#### test
Location: `ci/tasks/test.yml`

A task to test the service.

It can define dependent registry-images to help speed up build times.
These must be manually added in the pipeline, and updated against the service docker files.

As this task is self contained, we do not define an interface.

That is to say, we care that the task exists, we don't care what it does.

An example can be found [here][example-test].

#### lint
Location: `ci/tasks/lint.yml`

A task to lint the service.

It can define dependent registry-images to help speed up build times.
These must be manually added in the pipeline, and updated against the service docker files.

As this task is self contained, we do not define an interface.

That is to say, we care that the task exists, we don't care what it does.

An example can be found [here][example-lint].

### Tasks to implement for deploying

#### build-deployable
Location: `ci/tasks/build-deployable.yml`

A task to build a deployable service image.

The task must define its own runner.
Most likely, you will be using Concourse's [builder-task][builder-task], however if you
wish to use the GovWifi runner noted above, you can load it using this snippet:

```yml
image_resource:
  type: docker-image
    source:
      repository: "((readonly_private_ecr_repo_url))"
      tag: concourse-runner-latest
```

The task will be passed the params:
- TAG: 'staging' | 'production'
  denotes the tag to set on the image

The task must output to a folder called `image`.
The contents of the folder must include:

- `image.tar`
  The service image in OCI format
- `tag`
  Contains the tag name to set on the image when pushing.
  This **should** be the same as the `TAG` parameter.

An example can be found [here][example-build-deployable]

#### deploy
Location: `ci/tasks/deploy.yml`

A task to deploy the service after the image has been pushed.

It will be provided with these params:

- STAGE: 'staging' | 'production'
  denontes whether this is for staging or production
- AWS_ACCESS_KEY_ID:
- AWS_DEFAULT_REGION:
- AWS_SECRET_ACCESS_KEY:
  AWS Variables for use with `aws-cli`.
  They have access to reading/restarting ECS tasks/services, and pushing to ECR.

As part of this, a folder called `deploy-tools` will be loaded.
This will contain useful functions around the `aws-cli` to help keep duplicate code to a minimum.

[govwifi-runner]: https://github.com/govwifi-concourse-runner
[example-pre-build]: https://github.com/alphagov/govwifi-logging-api/blob/master/ci/tasks/pre-build.yml
[example-test]: https://github.com/alphagov/govwifi-logging-api/blob/master/ci/tasks/test.yml
[example-lint]: https://github.com/alphagov/govwifi-logging-api/blob/master/ci/tasks/lint.yml
[example-build-deployable]: ./example-tasks/build-deployable.yml
[example-deploy]: https://github.com/alphagov/govwifi-logging-api/blob/master/ci/tasks/deploy.yml
[builder-task]: https://github.com/concourse/builder-task
