# Backup a directory to an S3 bucket

This is a simple solution for backing up files in Kubernetes (from a
PersistentVolume or other filesystem location) to an S3 bucket (such
as provided by [Amazon S3][], [OpenStack Swift][], [Ceph][],
[MinIO][], and others).

[amazon s3]: https://aws.amazon.com/s3/
[openstack swift]: https://docs.openstack.org/swift/latest/
[ceph]: https://ceph.io/
[minio]: https://min.io/

This collection of manifests will deploy a [CronJob][] that
periodically runs a pod that backs up up a local (to the pod) directory
to an S3 bucket using the [MinIO Client][] (`mc`).

[minio client]: https://docs.min.io/docs/minio-client-complete-guide.html
[CronJob]: https://docs.openshift.com/container-platform/4.7/rest_api/workloads_apis/cronjob-batch-v1beta1.html

We're using the MinIO client because MinIO maintains a [container
image][] that we can use without modification. A similar solution
could be built using [s3cmd][], but there is no (maintained)
"official" s3cmd contianer image. The MinIO client is also
substantially simpler to configure in a containerized environment.

[container image]: https://hub.docker.com/r/minio/mc/
[s3cmd]: https://s3tools.org/s3cmd

## Requirements

In order to deploy these manifests, you need:

- The [OpenShift command line client][oc], `oc`
- [Kustomize][]

[oc]: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.7/
[kustomize]: https://kustomize.io/

## Configure

1. Copy `credentials-example.env` to `credentials.env`
   and update it with your bucket credentials. 

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

**WARNING** Make sure you don't add `credentials.env` to a Git
repository and accidentally expose your credentials.

## Deploy

To deploy the application to OpenShift, run:

```
kustomize build | oc apply -f-
```

To delete the application from OpenShift, run:

```
kustomize build | oc delete -f-
```
