# cfn-find-ami

Lambda function that returns the latest AMI that whose tags match a set
of inputs.  Useful as a CloudFormation custom resource.

This lambda function solves the problem of not knowing which AMI ID
to use when launching their application stack.

This project is launched as a CloudFormation stack, containing all the
necessary components to get started using this as a CloudFormation
custom resource in other stacks.

This project's CloudFormation stack creates an exported output that can
be imported by other stacks, to aid in resource discovery.  It is
suggested that this stack be launched in every region (where Lambda
is supported) so that the exported output can be used, as CloudFormation
does not support cross-region exported values.

## Goal

Provide a CloudFormation custom resource service endpoint that will
reliably find AMIs based on a tag/key lookup.

## Launching this stack

This project uses Ansible to laucnh the stack.  This is entirely optional
and the `cfn-template.yml` can be used separately if you want.  What
Ansible brings to the party is the ability to wait for the CloudFormation
stack to complete successfully, or fail with a non-zero exit if the stack
failed to launch.

```
$ ansible-playbook -i localhost.inventory -e 'environment_label=production'  create-stack.yml
```

The ability to specify `environment_label` means your CFN custom
resource can support updates and configuration tweaks without breaking
existing infrastructure.

## Using this Custom Resource

This lambda function can be used as a custom resource in another
CloudFormation stack.  Add the following in your template:

```
Resources:
  ... other resources ...

  FindAmiCustomResource:
    Type: Custom::FindAmiByProjectEnv
    Properties:
      Service:
      Project: !Ref ProjectName
      Environment: !Ref EnvironmentName
```

This assumes that you have two stack parameters: `ProjectName` and
`EnvironmentName`.

The result of the custom resource will be retrieved using the `Fn::GetAtt`
function:

```
  ... InstanceProfile definition ...
    ImageId: !GetAtt FindAmiCustomResource.ami_id
```

## Caveats / Things to Consider

* The Ansible playbook launches the stack in all regions where Lambda is
currently supported.
* The export naming scheme is `<service>:<resource>:<environment>:<component>`.  It may not match your naming schemes.  Fork this repo and change it.
* The Lambda function is provided inline, as this is the easiest way to
get a lambda stack running without extra steps.
* The Lambda function looks for tags on an AMI in an opinionated naming
scheme.

## License

tl;dr MIT License.

See [LICENSE](LICENSE) for more details.

