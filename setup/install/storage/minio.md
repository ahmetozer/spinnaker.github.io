---
layout: single
title:  "Minio"
sidebar:
  nav: setup
redirect_from: /setup/storage/minio/
---

> :warning: Losing Minio's data will mean losing all your Spinnaker
> application metadata, and configured pipelines.

[Minio](https://www.minio.io/){:target="\_blank"} is an S3-compatible object
store that you can host yourself. This is the persistent storage solution we
recommend when you don't want to depend on a cloud provider to host your
Spinnaker data.

## Prerequisites

Install Minio following the instructions on the [Minio
homepage](https://www.minio.io/){:target="\_blank"}, making sure to have it run
on an endpoint reachable by Spinnaker. Record the following values:

* `ENDPOINT`: The fully-qualifed endpoint Minio is reachable on. If Minio is
  running on the same machine as Spinnaker, this might be
  `http://127.0.0.1:9001` (note that using `localhost` instead of `127.0.0.1` doesn't work because the AWS SDK will then try to connect to http://<bucket-name>.localhost:9001).

* `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`: The access/secret keypair you've
  configured Minio with. These env vars need to be visible to the Minio process
  for them to work.

## Editing your storage settings

Given that Minio doesn't support versioning objects, we need to disable it
in Spinnaker. Add the following line to `~/.hal/$DEPLOYMENT/profiles/front50-local.yml`:

```yaml
spinnaker.s3.versioning: false
```

`$DEPLOYMENT` is typically `default`. Read more [here](/reference/halyard/#deployments). If this file does not exit, it may have to be created.

Run the following commands (notice we are picking S3 as our storage type,
because Minio implements the S3 API):

### Ubuntu installation

```bash
echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key # will be read on STDIN to avoid polluting your
                        # ~/.bash_history with a secret

hal config storage edit --type s3
```

### Docker container installation

```bash
# The next two lines should be run inside the docker container only
chcon -R --reference /root/.bashrc /root/.hal/
ls -lZa /root # Make sure the SELinux context is the same for all files/folders

echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key # will be read on STDIN to avoid polluting your
                        # ~/.bash_history with a secret

hal config storage edit --type s3
```

## Notes

> :warning: Unless you've configured Minio to [support virtual-host-style requests](https://docs.min.io/docs/minio-server-configuration-guide.html#Domain), you need to enable the path-style access mode: `hal config storage s3 edit --path-style-access true`. Otherwise, Front50 will fail with `java.net.UnknownHostException: $BUCKET_NAME.localhost` exceptions.  
> :warning: If you use Distributed installation and Front50 is in the crash loop, please set the endpoint as a Minio address instead of localhost.

## Next steps

After you've set up Minio as your external storage service, you're ready to
[deploy Spinnaker](/setup/install/deploy/).
