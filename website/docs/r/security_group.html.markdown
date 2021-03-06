---
layout: "aws"
page_title: "AWS: aws_security_group"
description: |-
  Provides a security group resource.
---

# Resource: aws_security_group

Provides a security group resource.

~> **NOTE on Security Groups and Security Group Rules:** Terraform currently
provides both a standalone [Security Group Rule resource](security_group_rule.html) (a single `ingress` or
`egress` rule), and a Security Group resource with `ingress` and `egress` rules
defined in-line. At this time you cannot use a Security Group with in-line rules
in conjunction with any Security Group Rule resources. Doing so will cause
a conflict of rule settings and will overwrite rules.

~> **NOTE:** Referencing Security Groups across VPC peering has certain restrictions. More information is available in the [VPC Peering User Guide](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-security-groups.html).

## Example Usage

Basic usage

```hcl
resource "aws_security_group" "allow_tls" {
  name        = "allow_tls"
  description = "Allow TLS inbound traffic"
  vpc_id      = "${aws_vpc.main.id}"

  ingress {
    # TLS (change to whatever ports you need)
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    # Please restrict your ingress to only necessary IPs and ports.
    # Opening to 0.0.0.0/0 can lead to security vulnerabilities.
    cidr_blocks = # add a CIDR block here
  }

  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
    prefix_list_ids = ["pl-12c4e678"]
  }
}
```

Basic usage with tags:

```hcl
resource "aws_security_group" "allow_tls" {
  name        = "allow_tls"
  description = "Allow TLS inbound traffic"

  ingress {
    # TLS (change to whatever ports you need)
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    # Please restrict your ingress to only necessary IPs and ports.
    # Opening to 0.0.0.0/0 can lead to security vulnerabilities.
    cidr_blocks = # add your IP address here
  }

  tags = {
    Name = "allow_all"
  }
}
```

## Argument Reference

The following arguments are supported:

* `name` - (Optional, Forces new resource) The name of the security group. If omitted, Terraform will
assign a random, unique name
* `name_prefix` - (Optional, Forces new resource) Creates a unique name beginning with the specified
  prefix. Conflicts with `name`.
* `description` - (Optional, Forces new resource) The security group description. Defaults to
  "Managed by Terraform". Cannot be "". __NOTE__: This field maps to the AWS
  `GroupDescription` attribute, for which there is no Update API. If you'd like
  to classify your security groups in a way that can be updated, use `tags`.
* `ingress` - (Optional) Can be specified multiple times for each
   ingress rule. Each ingress block supports fields documented below.
   This argument is processed in [attribute-as-blocks mode](/docs/configuration/attr-as-blocks.html).
* `egress` - (Optional, VPC only) Can be specified multiple times for each
      egress rule. Each egress block supports fields documented below.
      This argument is processed in [attribute-as-blocks mode](/docs/configuration/attr-as-blocks.html).
* `revoke_rules_on_delete` - (Optional) Instruct Terraform to revoke all of the
Security Groups attached ingress and egress rules before deleting the rule
itself. This is normally not needed, however certain AWS services such as
Elastic Map Reduce may automatically add required rules to security groups used
with the service, and those rules may contain a cyclic dependency that prevent
the security groups from being destroyed without removing the dependency first.
Default `false`
* `vpc_id` - (Optional, Forces new resource) The VPC ID.
* `tags` - (Optional) A mapping of tags to assign to the resource.

The `ingress` block supports:

* `cidr_blocks` - (Optional) List of CIDR blocks.
* `ipv6_cidr_blocks` - (Optional) List of IPv6 CIDR blocks.
* `prefix_list_ids` - (Optional) List of prefix list IDs.
* `from_port` - (Required) The start port (or ICMP type number if protocol is "icmp")
* `protocol` - (Required) The protocol. If you select a protocol of
"-1" (semantically equivalent to `"all"`, which is not a valid value here), you must specify a "from_port" and "to_port" equal to 0. If not icmp, tcp, udp, or "-1" use the [protocol number](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)
* `security_groups` - (Optional) List of security group Group Names if using
    EC2-Classic, or Group IDs if using a VPC.
* `self` - (Optional) If true, the security group itself will be added as
     a source to this ingress rule.
* `to_port` - (Required) The end range port (or ICMP code if protocol is "icmp").
* `description` - (Optional) Description of this ingress rule.

The `egress` block supports:

* `cidr_blocks` - (Optional) List of CIDR blocks.
* `ipv6_cidr_blocks` - (Optional) List of IPv6 CIDR blocks.
* `prefix_list_ids` - (Optional) List of prefix list IDs (for allowing access to VPC endpoints)
* `from_port` - (Required) The start port (or ICMP type number if protocol is "icmp")
* `protocol` - (Required) The protocol. If you select a protocol of
"-1" (semantically equivalent to `"all"`, which is not a valid value here), you must specify a "from_port" and "to_port" equal to 0. If not icmp, tcp, udp, or "-1" use the [protocol number](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)
* `security_groups` - (Optional) List of security group Group Names if using
    EC2-Classic, or Group IDs if using a VPC.
* `self` - (Optional) If true, the security group itself will be added as
     a source to this egress rule.
* `to_port` - (Required) The end range port (or ICMP code if protocol is "icmp").
* `description` - (Optional) Description of this egress rule.

~> **NOTE on Egress rules:** By default, AWS creates an `ALLOW ALL` egress rule when creating a
new Security Group inside of a VPC. When creating a new Security
Group inside a VPC, **Terraform will remove this default rule**, and require you
specifically re-create it if you desire that rule. We feel this leads to fewer
surprises in terms of controlling your egress rules. If you desire this rule to
be in place, you can use this `egress` block:

```hcl
egress {
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]
}
```

## Usage with prefix list IDs

Prefix list IDs are managed by AWS internally. Prefix list IDs
are associated with a prefix list name, or service name, that is linked to a specific region.
Prefix list IDs are exported on VPC Endpoints, so you can use this format:

```hcl
# ...
egress {
  from_port       = 0
  to_port         = 0
  protocol        = "-1"
  prefix_list_ids = ["${aws_vpc_endpoint.my_endpoint.prefix_list_id}"]
}

# ...
resource "aws_vpc_endpoint" "my_endpoint" {
  # ...
}
```

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `id` - The ID of the security group
* `arn` - The ARN of the security group
* `vpc_id` - The VPC ID.
* `owner_id` - The owner ID.
* `name` - The name of the security group
* `description` - The description of the security group
* `ingress` - The ingress rules. See above for more.
* `egress` - The egress rules. See above for more.

## Timeouts

`aws_security_group` provides the following [Timeouts](/docs/configuration/resources.html#timeouts)
configuration options:

- `create` - (Default `10 minutes`) How long to wait for a security group to be created.
- `delete` - (Default `10 minutes`) How long to wait for a security group to be deleted.

## Import

Security Groups can be imported using the `security group id`, e.g.

```
$ terraform import aws_security_group.elb_sg sg-903004f8
```
