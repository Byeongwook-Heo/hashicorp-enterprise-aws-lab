# HashiCorp Enterprise AWS Lab

HashiCorp Enterprise AWS 인프라 실습을 독립 저장소의 `main`에서 관리합니다. 기존 `factory-productivity-suite` 브랜치의 최신 인프라 변경을 포함하며, 애플리케이션은 [Vault Security Portal](https://github.com/Byeongwook-Heo/vault-security-portal)로 분리했습니다.

`envs/dev`가 통합 인프라의 시작점입니다. `aws-ec2-hcp`, `aws-ec2-hcp-hashicorp_lab`, `aws-ec2-tfe`는 기존 워크스페이스별 부트스트랩 예제입니다. HCP 예제 두 개는 워크스페이스 이름이 달라 별도로 보존했습니다.

실제 AMI·VPC·서브넷·보안그룹·키 페어는 Terraform 입력값으로 지정하세요. Bastion 접속 CIDR은 기본적으로 비어 있습니다. 아래 주소·리소스 ID는 익명화된 과거 예시이며 현재 배포 상태나 접속 주소를 의미하지 않습니다. 이번 저장소 분리에서는 AWS 리소스를 생성·변경하지 않았습니다.

Terraform code for a HashiCorp enterprise-style AWS lab environment.

## Current Status

The existing bootstrap EC2 instance has already been resized to `t4g.2xlarge`.

```text
Instance ID: i-00000000000000000
Public IP:   192.0.2.204
Private IP:  192.0.2.102
SSH:         ssh -i ~/.ssh/lab.pem ubuntu@192.0.2.204
```

The active HCP Terraform agent runs on this instance and is registered in the `aws-agent-pool` pool.

## Repository Layout

```text
envs/dev/                 Dev environment root module
modules/network/          VPC, subnets, routing, NAT gateways
modules/security/         Security groups for ALB, application, and RDS
modules/alb/              Application Load Balancer and target group
modules/compute/          Launch template and Auto Scaling Group
modules/data/             RDS PostgreSQL subnet group and instance
modules/iam/              EC2 instance profile for SSM and CloudWatch
modules/vault-enterprise/ Vault Enterprise Raft cluster
modules/keycloak/         Keycloak HA nodes, ALB, PostgreSQL, admin secret
modules/mcp-server/       Private MCP server, internal ALB, API Gateway VPC Link
modules/vault-benchmark-runner/
                          Private EC2 runner for Vault benchmark tests
docs/                     Operating notes
aws-ec2-*/                Earlier bootstrap and one-instance lab code
```

## Target Architecture

```text
Internet
  -> Public ALBs
  -> App Auto Scaling Group, EC2 t4g.2xlarge
  -> Keycloak Auto Scaling Group, EC2 t4g.2xlarge
  -> API Gateway HTTP API
  -> VPC Link
  -> Internal MCP ALB
  -> MCP Auto Scaling Group, EC2 t4g.2xlarge
  -> Vault Enterprise Raft cluster, 3 x EC2 t4g.2xlarge
  -> Vault benchmark runner, EC2 c7g.2xlarge
  -> PostgreSQL RDS Multi-AZ databases, db.t4g.2xlarge
```

The default design uses two Availability Zones, public subnets for public ALBs and NAT gateways, private application subnets for EC2 workloads, and isolated private database subnets for RDS.

## Workflows

Use `envs/dev` as the Terraform working directory.

```bash
cd envs/dev
terraform init
terraform fmt -recursive
terraform validate
terraform plan
```

The code is prepared for HCP Terraform organization `hashicorp_lab` and workspace `hashicorp_lab-enterprise-dev`.

## Secrets

Do not commit AWS credentials, `.tfvars`, Terraform state, private keys, or license files. Those are excluded by `.gitignore`.
