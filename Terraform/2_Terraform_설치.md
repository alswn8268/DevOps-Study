[Terraform 설치](https://developer.hashicorp.com/terraform/install)

```sh
[vagrant@ansible-controller ~]$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
[vagrant@ansible-controller ~]$ unzip awscliv2.zip
[vagrant@ansible-controller ~]$ sudo ./aws/install

[vagrant@ansible-controller ~]$ sudo yum install -y yum-utils
[vagrant@ansible-controller ~]$ sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
[vagrant@ansible-controller ~]$ sudo yum -y install terraform

[vagrant@ansible-controller ~]$ aws --version
aws-cli/2.32.11 Python/3.13.9 Linux/5.14.0-503.33.1.el9_5.x86_64 exe/x86_64.rocky.9
[vagrant@ansible-controller ~]$ terraform -v
Terraform v1.14.1
on linux_amd64
```

