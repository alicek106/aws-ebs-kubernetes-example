# Attaching EBS to Kubernetes cluster

This repository explains how to use EBS volume in Kubernetes cluster, by deploying NFS server container.

## Deploying EBS-attached NFS container in Kubernetes

It assumes that you are in Kubernetes master terminal, which can use ```kubectl```.

------

1. If you didn't install awscli, install first.

```
$ pip install virtualenv && \
  virtualenv awscli-env && \
  source awscli-env/bin/activate
```

```
$ pip install awscli && aws --version
aws-cli/1.16.131 Python/2.7.12 Linux/4.4.0-1075-aws botocore/1.12.121
```

2. Set bash variable for AWS access. It is acquired in AWS IAM role management.

```
$ export AWS_ACCESS_KEY_ID=<ACCESS_KEY>
$ export AWS_SECRET_ACCESS_KEY=<SECRET_KEY>
```

3. Check the region of Kubernetes cluster. Below example shows all Kubernetes instances are launched in ```ap-northeast-2a``` region.

```
(awscli-env) $ aws ec2 describe-instances --region ap-northeast-2 | grep AvailabilityZone
                        "AvailabilityZone": "ap-northeast-2a"
                        "AvailabilityZone": "ap-northeast-2a"
                        "AvailabilityZone": "ap-northeast-2a"
                        "AvailabilityZone": "ap-northeast-2a"
```

4. Create EBS volume. ```--size``` parameter means size of volume (GB). Confirm that you are using right region for ```--region``` parameter corresponding to Kubernetes EC2 instances.

```
(awscli-env) $ export VOLUME_ID=$(aws ec2 create-volume --size 1 \
  --region ap-northeast-2 \
  --availability-zone ap-northeast-2a \
  --volume-type gp2 | jq '.VolumeId' -r)
```

5. Create NFS deployment. That's all.

```
(awscli-env) $ cat NFS-aws-ebs.yaml | sed "s/{{VOLUME_ID}}/$VOLUME_ID/g" | kubectl apply -f -
```



## Troubleshooting

If your worker EC2 instanses don't have appropriate IAM role to use EBS, it will fail to mount EBS.
