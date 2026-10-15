# CloudFormation Nested Stack Template : Load Balancer

<!-- Row 1: Status - Most Important -->
[![Release](https://img.shields.io/github/v/release/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer?label=Release)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/releases)&nbsp;[![Release Workflow](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/actions/workflows/release.yaml)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/badge/Languages-YAML-blue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)](https://github.com/subhamay-bhattacharyya/cfn-nested-aws-load-balancert)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-load-balancer/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/b5b980ffe2f8de29b7aff6443fe8f4ae/raw/cfn-nested-aws-load-balancert.json)](https://gist.github.com/subhamay-bhattacharyya/b5b980ffe2f8de29b7aff6443fe8f4ae)

---

A nested CloudFormation template that creates an Application, Network, or Gateway Elastic Load Balancer (ELBv2), with an optional listener. When the template is run as part of a CI build, it also creates its own VPC, internet gateway, subnets, web security group, and target group so the load balancer can be tested in isolation.

## Overview

`cloudformation/template.yaml` is a **nested stack template**. Parent stacks invoke it with `AWS::CloudFormation::Stack` and `TemplateURL`, passing the load balancer settings as parameters and reading the load balancer details back from its outputs.

The template can also be deployed directly with `aws cloudformation deploy`, as shown in the usage section below.

## Template Features

- ✅ Application (ALB), Network (NLB), and Gateway (GWLB) load balancers
- ✅ Internal or internet-facing scheme, with IPv4 or dualstack addressing
- ✅ Optional listener, created only when `ListenerProtocol` is set
- ✅ Optional TLS certificate and SSL policy for HTTPS and TLS listeners
- ✅ Security groups attached to application and network load balancers, skipped for gateway load balancers, which do not support them
- ✅ Parameters grouped and labeled in the CloudFormation console
- ✅ CI mode: when `CiSuffix` is set, creates a VPC, two subnets, an internet gateway with a public route table, a web security group, and a target group, then builds the load balancer in them

### CI Mode

Setting `CiSuffix` to a non-empty value turns on CI mode. The template then:

| Resource | Purpose |
| -------- | ------- |
| `CiVpc` | `10.0.0.0/16` VPC with DNS support and hostnames |
| `CiSubnetA`, `CiSubnetB` | `10.0.0.0/24` and `10.0.1.0/24`, in the first two availability zones |
| `CiInternetGateway`, `CiInternetGatewayAttachment` | Internet access for the VPC |
| `CiRouteTable`, `CiDefaultRoute`, `CiSubnet*RouteTableAssociation` | Public route table (`0.0.0.0/0` to the internet gateway) for both subnets |
| `CiWebSecurityGroup` | Allows inbound TCP 80 and 443 from `0.0.0.0/0` |
| `CiTargetGroup` | Target group matching the load balancer type (see below) |

In CI mode:

- The load balancer uses the CI subnets, so `SubnetIds` is ignored.
- Application and network load balancers use `CiWebSecurityGroup`, so `SecurityGroupIds` is ignored. Gateway load balancers get no security group.
- The listener's default action forwards to `CiTargetGroup`, so `DefaultTargetGroupArn` is ignored.

The CI target group uses these protocols and ports:

| `LoadBalancerType` | Target group protocol | Port |
| ------------------ | --------------------- | ---- |
| `application` | HTTP | 80 |
| `network` | TCP | 80 |
| `gateway` | GENEVE | 6081 |

## Parameters

### Common Configuration

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `ProjectName` | String | `cfn` | Project name, 1–12 lowercase letters, numbers, or hyphens |
| `Environment` | String | `devl` | One of `devl`, `test`, `prod` |
| `CiSuffix` | String | `""` | Optional CI suffix, up to 4 characters. A non-empty value enables CI mode and is appended to the load balancer name |

### Load Balancer Configuration

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `LoadBalancerBaseName` | String | `test-alb` | Short name, 1–10 lowercase letters, numbers, or hyphens |
| `LoadBalancerType` | String | `application` | One of `application`, `network`, `gateway` |
| `Scheme` | String | `internal` | `internal` or `internet-facing` |
| `IpAddressType` | String | `ipv4` | `ipv4` or `dualstack` |

### Network Configuration

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `SubnetIds` | CommaDelimitedList | `subnet-00000000` (placeholder) | Subnets in at least two AZs. Ignored in CI mode. Pass real subnets when `CiSuffix` is empty |
| `SecurityGroupIds` | CommaDelimitedList | `""` | Security groups for application and network load balancers. Leave blank for gateway load balancers. Ignored in CI mode |

### Listener Configuration

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `ListenerProtocol` | String | `HTTP` | `HTTP`, `HTTPS` (application); `TCP`, `UDP`, `TCP_UDP`, `TLS` (network); `GENEVE` (gateway). Blank skips the listener |
| `ListenerPort` | Number | `80` | Listener port, 1–65535. Used only when `ListenerProtocol` is set. Use 443 for HTTPS and 6081 for GENEVE |
| `DefaultTargetGroupArn` | String | `""` | Target group the listener forwards to. Required when `ListenerProtocol` is set outside CI mode |
| `CertificateArn` | String | `""` | ACM certificate for HTTPS and TLS listeners |
| `SslPolicy` | String | `ELBSecurityPolicy-TLS13-1-2-2021-06` | SSL policy, used only when `CertificateArn` is set |

## Outputs

| Output | Description |
| ------ | ----------- |
| `LoadBalancerArn` | ARN of the load balancer |
| `LoadBalancerFullName` | Full name of the load balancer, as used in CloudWatch metrics |
| `DnsName` | DNS name of the load balancer |
| `CanonicalHostedZoneId` | Hosted zone ID for Route 53 alias records |
| `ListenerArn` | ARN of the listener. Present only when a listener is created |

## Usage

### 1. Reference from a Parent Stack

```yaml
LoadBalancerNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://your-cfn-templates-bucket.s3.us-east-1.amazonaws.com/cfn-nested-aws-load-balancer/template.yaml
    Parameters:
      ProjectName: myproject
      Environment: devl
      LoadBalancerBaseName: alb
      LoadBalancerType: application
      Scheme: internal
      SubnetIds: !Join [",", [subnet-0123456789abcdef0, subnet-0123456789abcdef1]]
      SecurityGroupIds: !Ref AlbSecurityGroupId
      ListenerProtocol: HTTP
      ListenerPort: 80
      DefaultTargetGroupArn: !Ref AppTargetGroupArn
      CiSuffix: !Ref CiSuffix
    Tags:
      - Key: Environment
        Value: devl

Outputs:
  LoadBalancerArn:
    Value: !GetAtt LoadBalancerNestedStack.Outputs.LoadBalancerArn
  DnsName:
    Value: !GetAtt LoadBalancerNestedStack.Outputs.DnsName
```

Upload `cloudformation/template.yaml` to S3 before referencing it.

### 2. Deploy Directly with the AWS CLI

Validate the template:

```bash
aws cloudformation validate-template \
  --template-body file://cloudformation/template.yaml
```

Deploy an internal application load balancer with an HTTP listener in an existing VPC:

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name cfn-nested-aws-load-balancer-devl \
  --parameter-overrides \
    ProjectName=myproject \
    Environment=devl \
    LoadBalancerBaseName=alb \
    SubnetIds=subnet-0123456789abcdef0,subnet-0123456789abcdef1 \
    DefaultTargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/app/0123456789abcdef \
  --region us-east-1
```

Deploy in CI mode, which creates its own network. `SubnetIds` can be omitted, because it defaults to a placeholder that CI mode ignores:

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name cfn-nested-aws-load-balancer-ci \
  --parameter-overrides \
    ProjectName=myproject \
    Environment=devl \
    LoadBalancerBaseName=alb \
    CiSuffix=$CI_PIPELINE_ID \
    Scheme=internal \
  --region us-east-1
```

Use `Scheme=internal` in CI mode unless you need internet-facing traffic. The default route to the internet gateway is created with the other CI resources, but the load balancer does not wait for it, so an internet-facing load balancer can occasionally fail to create during the first deployment.

## Load Balancer Naming

Without a CI suffix:

```bash
{ProjectName}-{Environment}-{LoadBalancerBaseName}
```

Example: `myproject-devl-alb`

With a CI suffix:

```bash
{ProjectName}-{Environment}-{LoadBalancerBaseName}-{CiSuffix}
```

Example: `myproject-devl-alb-1234`

Load balancer names are limited to 32 characters. Keep `ProjectName`, `LoadBalancerBaseName`, and `CiSuffix` short enough to stay within that limit.

## Best Practices Implemented

- ✅ Parameters validated with allowed patterns and value ranges
- ✅ Load balancer names derived from project, environment, and base name for consistency
- ✅ Gateway load balancers created without security groups, as AWS requires
- ✅ Optional listener and TLS settings, so the template covers HTTP-only and HTTPS deployments
- ✅ Outputs returned to parent stacks through `!GetAtt`

## License

MIT
