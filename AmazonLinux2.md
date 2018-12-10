# Amazon Linux 2

- [Migrating to Amazon Linux 2](https://cloudonaut.io/migrating-to-amazon-linux-2/)


## Amazon Linux 2 for ECS

- CloudFormation

- Add `Parameters` for always using the latest AMI

    ```
    "EcsOptimizedAmi": {
      "Description": "Amazon ECS-optimized AMI",
      "Type": "AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>",
      "Default": "/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image_id"
    },

    ```    

- In `BlockDeviceMappings`, use `/dev/xvda` instead of `xvdcz`

    ```
        "BlockDeviceMappings": [{
          "DeviceName": "/dev/xvda",
          "Ebs": {
            "VolumeType": "gp2",
            "VolumeSize": {"Ref": "EcsInstanceVolumeSize"}
          }
        }],
    ```

- In `AWS::CloudFormation::Init`, use `awslogsd` instead of `awslogs`

    ```
            "services": {
              "sysvinit": {
                "cfn-hup": {
                  "enabled": true,
                  "ensureRunning": true,
                  "files": [
                    "/etc/cfn/cfn-hup.conf",
                    "/etc/cfn/hooks.d/cfn-auto-reloader.conf"
                  ]
                },
                "awslogsd": {
                  "enabled": true,
                  "ensureRunning": true,
                  "files": [
                    "/etc/awslogs/awslogs.conf",
                    "/etc/awslogs/awscli.conf"
                  ]
                }
              }
            }
    ```