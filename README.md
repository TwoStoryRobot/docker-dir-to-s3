# docker-dir-to-s3

Docker container that performs daily backups from a local directory to S3

# How to Use

Basic usage is as follows:

```
docker run \
  -e "AWS_BUCKET=my-container" \
  -e "AWS_ACCESS_KEY_ID=MYACCESSKEYID" \
  -e "AWS_SECRET_ACCESS_KEY=3jk2kj3lkll+EXAMPLE/k213jl12k3kj213lkj213ll" \
  -e "AWS_DEFAULT_REGION=ca-central-1" \
  -v /my_local_dir/:/upload/:ro
  twostoryrobot/dir-to-s3
```

This will start cron in the background which will run an upload script daily.
Any volumes that you mount to the container's `/upload/` directory will be 
compressed into a tar archive using bzip compression. The resulting file is
uploaded to S3 with filename `<timestamp>.tar.gz`.

You can mount a single volume from another container or from your filesystem 
to the root of the `/upload/` directory, or you can also backup multiple volumes
by mounting them as subdirectories on `/upload/dir1/`, `/upload/dir2/`, and so 
on.

# Security Best Practices

To perform the backup, only one permission is needed: `s3:PutObject`. You should
create a separate Managed Policy which can be assigned to a user with this 
permission. Additionally, you should limit the permission to only the bucket
you plan to upload to. Your policy will look something similar to this:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1291030123918",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::my-container/*"
            ]
        }
    ]
}
```

You should create a separate user in your AWS IAM console to perform the 
backups. This user does not need a password, so they should only have the 
'Programmatic access' Access Type. AWS will provide a `AWS_ACCESS_KEY_ID` and
`AWS_SECRET_ACCESS_KEY` for that user. Assign the Managed Policy that you 
created to this user, and provide no other permissions to it.

In the event that your container or host system is compromised, an attacker will
have a key which can only write output to the container and cannot read the 
backups or perform any other operations on your AWS account. They could 
potentially use this to overwrite files, so make sure you enable versioning on
the bucket as well.

It's also good practice to only mount your volumes as read-only (`:ro`), since
this container does not need to write to any volumes.
