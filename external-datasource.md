# External Data Source
If all else fails, run a script to extract the data.

#### Your TF code
```hcl
data "external" "cluster_lb_arn" {
  depends_on = [helm_release.application_api]

  program = ["sh", "${path.module}/aws_cli.sh"]

  query = {
    vpc = module.vpc.vpc_id
  }
}
```

#### aws_cli.sh
```
##! /bin/bash
eval "$(jq -r '@sh "VPC=\(.vpc)"')"
aws elbv2 describe-load-balancers --region us-east-2 --output json | jq --arg vpc $VPC -r '.LoadBalancers[] | select(.VpcId == $vpc) |  {"Result":.LoadBalancerArn}'
```

## Common Issues
- you need to accept and return a json object
```bash
//to test:
echo '{"vpc":"vpc-xxxx"}' | ./aws_cli.sh
```
- when working with windows, fix the new line characters, this fucked me up big time
```bash
dos2unix aws_cli.sh
```
