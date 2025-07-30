---
layout: wiki 
title: Terraform
tags:  ["Infrastructure"]
last_modified_at: 2022/05/16 17:52:10
---

<!-- TOC -->

- [인증](#인증)
  - [tf import](#tf-import)
- [AWS](#aws)

<!-- /TOC -->

# 인증
서비스 계정을 만들고 JSON 인증으로 공통으로 사용할 수 있지만 혼자서 쓸때는 `credentials`를 설정하지 않으면 `gcloud` 개인 인증으로 사용한다.

provider에 따라 aws와 gcp를 자동으로 인식한다.

## tf import
실제 서버 구성을 가져와서 tf.state에 저장한다. 이 내용을 기준으로 main.tf를 만들 수 있다.

# AWS
```
// main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9"
}

provider "aws" {
  profile = "user1"
  region  = "ap-northeast-2"
}

locals {
  serverconfig = [
    for server in var.configuration : [
      for i in range(1, server.no_of_instances + 1) : {
        instance_name = server.application_name == "skpark-login" || server.application_name == "skpark-install" ? "${server.application_name}" : "${server.application_name}${i}"
        instance_type = server.instance_type
        volume_size = server.volume_size
      }
    ]
  ]
}
// We need to Flatten it before using it
locals {
  instances = flatten(local.serverconfig)
}

resource "aws_instance" "infra1" {
  for_each = { for server in local.instances : server.instance_name => server }

  ami                         = "ami-0ded4aeabdfcffac4" // Ubuntu - Deep Learning AMI
  associate_public_ip_address = true
  instance_type               = each.value.instance_type
  key_name                    = "key1"
  security_groups = [
    "sg-XXXX"
  ]
  subnet_id = "subnet-XXXX"
  tags = {
    Name = each.value.instance_name
  }
  root_block_device {
    volume_size = each.value.volume_size
  }
}

// variables.tf
variable "configuration" {
  description = "The total configuration, List of Objects/Dictionary"
  default     = [{}]
}

// terraform.tfvars
configuration = [
  {
    "application_name" : "skpark-login",
    "instance_type" : "i3en.xlarge",
    "no_of_instances" : "1",
    "volume_size": 2000 // 2TB
  },
  {
    "application_name" : "skpark-install",
    "instance_type" : "t3.xlarge",
    "no_of_instances" : "1",
    "volume_size": 200 
  },
  {
    "application_name" : "skpark-gpu-node",
    "instance_type" : "g4dn.xlarge", # GPU(T4 1ea) nodes
    "no_of_instances" : "2"
    "volume_size": 200
  }
]
```

`provider` 정보에 따라 해당 profile 인증을 사용하고, `$ terraform init`으로 현재 인프라 구성 상태 저장, `$ terraform apply -auto-approve`로 인프라를 provisioning 한다.

현재 상태 조회는 `$ terraform state list`, 삭제는 `$ terraform destroy`