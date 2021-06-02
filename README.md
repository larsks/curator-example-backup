# Backup a directory to an S3 bucket

## Overview

This collection of manifests will deploy a [CronJob][] that
periodically runs a pod that backs up up a local (to the pod) directory
to an S3 bucket.

[CronJob]: https://docs.openshift.com/container-platform/4.7/rest_api/workloads_apis/cronjob-batch-v1beta1.html

## Configuration

1. Copy `credentials-example.env` to `credentials.env`
   and update it with your AWS key id and secret.

2. Update `config.env`.

    - `BACKUP_SRC` is the path to the directory you want to back up
    - `BACKUP_DST` is the bucket name and path for backup destination
    - `S3_ENDPOINT` is your S3 API endpoint

3. Review the volume configuration in [`cronjob.yaml`](cronjob.yaml)
   and make sure you are mounting the correct volume and that the
   mountpoint corresponds to your `BACKUP_SRC` setting.

4. Deploy the application to OpenShift.

    - Run `oc apply -k .` to deploy this application into the
      namespace defined by the `namespace:` setting in
      `kustomization.yaml`.

**WARNING** Make sure you don't add `credentials.env` to a Git
repository and accidentally expose your credentials.
