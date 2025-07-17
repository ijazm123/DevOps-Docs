Terraform Collection Types Guide

This guide explains the difference between:

\- map(string)

\- list(string)

\- map(object(...))

\- list(object(...))

with examples, use cases, and a comparison table.

\------------------------------------------------------------

1\. map(string)

\- Structure: Key-value pairs with string values.

\- Access: var.my\_map\["key"\]

\- Use Case: Tags, key-value configurations.

Example:

variable "tags" {

type = map(string)

default = {

Environment = "dev"

Owner = "teamA"

}

}

output "env\_tag" {

value = var.tags\["Environment"\]

}

\------------------------------------------------------------

2\. list(string)

\- Structure: Ordered list of strings.

\- Access: var.my\_list\[0\]

\- Use Case: Subnets, availability zones, simple arrays.

Example:

variable "subnets" {

type = list(string)

default = \["subnet-1", "subnet-2", "subnet-3"\]

}

output "first\_subnet" {

value = var.subnets\[0\]

}

\------------------------------------------------------------

3\. map(object(...))

\- Structure: A map where values are objects with multiple attributes.

\- Access: var.my\_map\["key"\].attribute

\- Use Case: Multiple resources with structured attributes (e.g., VMs).

Example:

variable "vms" {

type = map(object({

size = string

location = string

os = string

}))

default = {

vm1 = {

size = "Standard\_B2s"

location = "East US"

os = "linux"

}

vm2 = {

size = "Standard\_D2s\_v3"

location = "West US"

os = "windows"

}

}

}

resource "azurerm\_linux\_virtual\_machine" "example" {

for\_each = var.vms

name = each.key

size = each.value.size

location = each.value.location

}

\------------------------------------------------------------

4\. list(object(...))

\- Structure: A list of objects with multiple attributes.

\- Access: var.my\_list\[0\].attribute

\- Use Case: Ordered configurations like firewall rules, policies.

Example:

variable "firewall\_rules" {

type = list(object({

name = string

priority = number

destination = string

}))

default = \[

{

name = "rule1"

priority = 100

destination = "10.0.1.0/24"

},

{

name = "rule2"

priority = 200

destination = "10.0.2.0/24"

}

\]

}

output "first\_rule\_name" {

value = var.firewall\_rules\[0\].name

}

\------------------------------------------------------------

Comparison Table:

Type | Structure | Access Method | Use Case

\-----------------------------------------------------------------------------------------

map(string) | Key-value strings | var.map\["key"\] | Tags, configs

list(string) | Ordered strings | var.list\[0\] | Subnets, availability zones

map(object(...)) | Map of objects | var.map\["key"\].attr | Multiple VMs, structured resources

list(object(...)) | List of objects | var.list\[0\].attr | Firewall rules, ordered configs

Rule of Thumb:

\- Use map(string) when you need lookup by key and simple values.

\- Use list(string) when order matters and values are simple.

\- Use map(object) when you need multiple structured attributes per key.

\- Use list(object) when you need ordered structured data.