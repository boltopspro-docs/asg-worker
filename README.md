<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/asg-worker/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# AutoScaling Worker Blueprint

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiNitGYjRkbUdQZy8xSGZhWVc2SFc1ZzZ1Q09JeW1YUDVkYUFHZ2dxS3E2YnRuUkJodzFVeUFMMW1kS0hRVlh4RDVEbHlBNGpYNVRVWlhJWXBFTFJBZzVjPSIsIml2UGFyYW1ldGVyU3BlYyI6ImhCRCtqb28vcFJmS2pGVmoiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

This blueprint provisions an AutoScaling fleet of instances. It can be used as a worker tier among other use cases.

* Several [AWS::AutoScaling::AutoScalingGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-group.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The blueprint is extremely flexible and configurable for your needs.
* Template makes use of the flexible [AWS::AutoScaling::AutoScalingGroup MixedInstancesPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-autoscaling-autoscalinggroup-mixedinstancespolicy.html). This allows you to use spot, on-demand, or mixed instance types. For example, you can run 70% spot and 30% on-demand instances.  Set `OnDemandPercentageAboveBaseCapacity=0` to run all spot instances.
* You can launch instances in a Custom VPC and Subnet by configuring `VPCZoneIdentifier` and `VpcId`.
* You can customize the UserData script and control the bootstrap process with a `@user_data_script` variable.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/asg-worker values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "asg-worker", git: "git@github.com:boltopspro/asg-worker.git"
```

## Configure

First you want to configure the [configs](https://lono.cloud/docs/core/configs/) files. Use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed asg-worker

To deploy to additional environments:

    LONO_ENV=production  lono seed asg-worker

The generated files in `config/asg-worker` folder look something like this:

    configs/asg-worker/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Here's a params example.

configs/asg-worker/params/development.txt:

    # Parameter Group: AWS::AutoScaling::AutoScalingGroup
    VPCZoneIdentifier=<%= lookup_output("vpc.PrivateAppSubnets") %> # (required)
    # MaxSize=8
    # MinSize=1

    # Parameter Group: AWS::AutoScaling::AutoScalingGroup MixedInstancesPolicy InstancesDistribution
    OnDemandPercentageAboveBaseCapacity=0 # 0 means all spot instances, 100 means all on-demand instances. Default is 100
    # OnDemandBaseCapacity= # The minimum amount of the Auto Scaling group's capacity that must be fulfilled by On-Demand Instances.
    # SpotAllocationStrategy= # lowest-price | capacity-optimized
    # SpotInstancePools= # The range is 1 to 20. Higher means more spot pools.

    # Parameter Group: AWS::EC2::LaunchTemplate LaunchTemplateData
    # InstanceType=t3.micro
    # KeyName=

    # Parameter Group: AWS::EC2::SecurityGroup
    VpcId=<%= lookup_output("vpc.PrivateAppSubnets") %> # Required. Though only used if SecurityGroupIds not set to create Security Group. # (required)

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy asg-worker-development --blueprint asg-worker --sure
    LONO_ENV=production  lono cfn deploy asg-worker-production  --blueprint asg-worker --sure


### Customzing User Data

To customer the UserData launch script to use, use the `@user_data_script` variable. Example:

configs/asg-worker/variables/development.rb:

```ruby
@user_data_script = "configs/asg-worker/user_data/bootstrap.sh"
```

Here's an example `bootstrap.sh` script:

    #!/bin/bash
    uptime
