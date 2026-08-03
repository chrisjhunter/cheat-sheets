# Terraform Cheat Sheet

## Core workflow
```bash
terraform init                       # download providers/modules, set up backend
terraform init -upgrade                # upgrade providers to latest allowed versions
terraform validate                       # check syntax/config validity
terraform fmt                              # auto-format .tf files
terraform fmt -recursive                     # format all subdirs
terraform plan                                 # preview changes
terraform plan -out=plan.tfplan                  # save plan to apply later
terraform apply                                    # apply changes (prompts to confirm)
terraform apply plan.tfplan                          # apply a saved plan (no prompt)
terraform apply -auto-approve                          # skip confirmation (careful)
terraform destroy                                        # tear down all managed resources
```

## State
```bash
terraform state list                       # list resources in state
terraform state show aws_instance.web        # show attributes of one resource
terraform state mv aws_instance.a aws_instance.b   # rename resource in state
terraform state rm aws_instance.web            # remove from state (doesn't destroy real resource)
terraform state pull > state.json                 # dump raw state
terraform import aws_instance.web i-0123456789      # bring existing infra under management
```

## Workspaces
```bash
terraform workspace list
terraform workspace new staging
terraform workspace select staging
terraform workspace show
```

## Variables
```bash
terraform plan -var="instance_type=t3.micro"
terraform plan -var-file="prod.tfvars"
export TF_VAR_instance_type=t3.micro          # env var equivalent
```
```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

## Targeting & scoping
```bash
terraform plan -target=aws_instance.web        # limit to a specific resource
terraform apply -target=module.vpc
terraform plan -destroy                          # preview a destroy without applying
terraform refresh                                   # sync state with real infra (deprecated in favor of plan -refresh-only)
terraform plan -refresh-only
```

## Debugging
```bash
TF_LOG=DEBUG terraform apply                  # verbose logging
TF_LOG=DEBUG TF_LOG_PATH=tf.log terraform apply   # log to file
terraform graph | dot -Tpng > graph.png         # visualize dependency graph
terraform console                                 # interactive expression evaluator
```

## Modules
```bash
terraform get                          # download/update modules
terraform get -update
```
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  cidr    = "10.0.0.0/16"
}
```

## Output values
```bash
terraform output                    # show all outputs
terraform output vpc_id               # show one output
terraform output -json                  # machine-readable
```

## Useful one-liners
```bash
terraform plan | grep -A2 "will be destroyed"        # find destructive changes at a glance
terraform show -json plan.tfplan | jq '.resource_changes[].change.actions'  # summarize action types
terraform providers                                    # list required providers
terraform version                                         # TF + provider versions
find . -name "*.tf" | xargs terraform fmt -check           # CI check for unformatted files
```
