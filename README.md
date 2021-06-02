# Backup a directory to an S3 bucket

## Overview

This collection of manifests will deploy a [CronJob][] that
periodically runs a pod that backs up up a local (to the pod) directory
to an S3 bucket.

We use `mc` (the [MinIO Client][]) to interface with the remote S3 endpoint.
We're using `mc` rather than, say, [s3cmd][] because:

- `mc` is easy to configure
- MinIO maintains a [docker image][] that we can use without any modification.

[minio client]: https://docs.min.io/docs/minio-client-complete-guide.html
[CronJob]: https://docs.openshift.com/container-platform/4.7/rest_api/workloads_apis/cronjob-batch-v1beta1.html
[docker image]: https://hub.docker.com/r/minio/mc/
[s3cmd]: https://s3tools.org/s3cmd

## Configuration

1. Copy `credentials-example.env` to `credentials.env`
   and update it with your AWS key id and secret.

2. Update `config.env`. The following configuration variables are
   required:

     - `BACKUP_SRC` -- the path to the directory you want to back up
     - `BACKUP_DST` -- the bucket name and path for backup destination
     - `S3_ENDPOINT` -- your S3 API endpoint

   Optionally, you may set:

     - `MC_GLOBAL_FLAGS` -- flags passed to all invocations of the
       `mc` command
     - `MC_MIRROR_FLAGS` -- flags passed only to the `mc mirror`
       command

   By default, `mc mirror` will not overwrite files that exist in the
   destination, even if they have been modified locally. Set
   `MC_MIRROR_FLAGS=--overwrite` to permit `mc` to overwrite remote
   files.

   For more information, see the [documentation for the `mirror`
   command][mirror-doc].

   [mirror-doc]: https://docs.min.io/docs/minio-client-complete-guide.html#mirror

3. Review the volume configuration in [`cronjob.yaml`](cronjob.yaml)
   and make sure you are mounting the correct volume and that the
   mountpoint corresponds to your `BACKUP_SRC` setting.

4. Deploy the application to OpenShift.

    - Run `oc apply -k .` to deploy this application into the
      namespace defined by the `namespace:` setting in
      `kustomization.yaml`.

**WARNING** Make sure you don't add `credentials.env` to a Git
repository and accidentally expose your credentials.
