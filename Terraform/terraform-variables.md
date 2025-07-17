# Terraform Collection Types Guide

This guide explains the difference between Terraform's main collection types with examples, use cases, and a comparison table.

## Collection Types Overview

- **`map(string)`** - Key-value pairs with string values
- **`list(string)`** - Ordered list of strings  
- **`map(object(...))`** - Map where values are objects with multiple attributes
- **`list(object(...))`** - List of objects with multiple attributes

---

## 1. `map(string)`

### Structure
Key-value pairs with string values.

### Access Method
```hcl
var.my_map["key"]
```

### Use Case
Tags, key-value configurations, environment-specific settings.

### Example
```hcl
variable "tags" {
  type = map(string)
  default = {
    Environment = "dev"
    Owner       = "teamA"
    Project     = "terraform-demo"
  }
}

output "env_tag" {
  value = var.tags["Environment"]
}

# Usage in resource
resource "azurerm_resource_group" "example" {
  name     = "rg-example"
  location = "East US"
  tags     = var.tags
}
```

---

## 2. `list(string)`

### Structure
Ordered list of strings.

### Access Method
```hcl
var.my_list[index]
```

### Use Case
Subnets, availability zones, simple arrays where order matters.

### Example
```hcl
variable "subnets" {
  type = list(string)
  default = ["subnet-1", "subnet-2", "subnet-3"]
}

variable "availability_zones" {
  type = list(string)
  default = ["eastus-1", "eastus-2", "eastus-3"]
}

output "first_subnet" {
  value = var.subnets[0]
}

# Usage with count
resource "azurerm_subnet" "example" {
  count                = length(var.subnets)
  name                 = var.subnets[count.index]
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.${count.index + 1}.0/24"]
}
```

---

## 3. `map(object(...))`

### Structure
A map where values are objects with multiple attributes.

### Access Method
```hcl
var.my_map["key"].attribute
```

### Use Case
Multiple resources with structured attributes (e.g., VMs, databases, storage accounts).

### Example
```hcl
variable "vms" {
  type = map(object({
    size     = string
    location = string
    os       = string
    disk_size = number
  }))
  default = {
    web-server = {
      size      = "Standard_B2s"
      location  = "East US"
      os        = "linux"
      disk_size = 64
    }
    db-server = {
      size      = "Standard_D2s_v3"
      location  = "West US"
      os        = "windows"
      disk_size = 128
    }
    app-server = {
      size      = "Standard_B1s"
      location  = "Central US"
      os        = "linux"
      disk_size = 32
    }
  }
}

# Usage with for_each
resource "azurerm_linux_virtual_machine" "linux_vms" {
  for_each = {
    for k, v in var.vms : k => v
    if v.os == "linux"
  }
  
  name                = each.key
  size                = each.value.size
  location            = each.value.location
  resource_group_name = azurerm_resource_group.example.name
  
  # ... other configuration
}

output "vm_details" {
  value = {
    for k, v in var.vms : k => {
      size     = v.size
      location = v.location
    }
  }
}
```

---

## 4. `list(object(...))`

### Structure
A list of objects with multiple attributes.

### Access Method
```hcl
var.my_list[index].attribute
```

### Use Case
Ordered configurations like firewall rules, policies, network security rules.

### Example
```hcl
variable "firewall_rules" {
  type = list(object({
    name        = string
    priority    = number
    direction   = string
    access      = string
    protocol    = string
    source_port = string
    destination_port = string
    destination = string
  }))
  default = [
    {
      name             = "allow-ssh"
      priority         = 100
      direction        = "Inbound"
      access           = "Allow"
      protocol         = "Tcp"
      source_port      = "*"
      destination_port = "22"
      destination      = "10.0.1.0/24"
    },
    {
      name             = "allow-http"
      priority         = 200
      direction        = "Inbound"
      access           = "Allow"
      protocol         = "Tcp"
      source_port      = "*"
      destination_port = "80"
      destination      = "10.0.2.0/24"
    },
    {
      name             = "deny-all"
      priority         = 4096
      direction        = "Inbound"
      access           = "Deny"
      protocol         = "*"
      source_port      = "*"
      destination_port = "*"
      destination      = "*"
    }
  ]
}

# Usage with count (order matters!)
resource "azurerm_network_security_rule" "example" {
  count = length(var.firewall_rules)
  
  name                       = var.firewall_rules[count.index].name
  priority                   = var.firewall_rules[count.index].priority
  direction                  = var.firewall_rules[count.index].direction
  access                     = var.firewall_rules[count.index].access
  protocol                   = var.firewall_rules[count.index].protocol
  source_port_range          = var.firewall_rules[count.index].source_port
  destination_port_range     = var.firewall_rules[count.index].destination_port
  destination_address_prefix = var.firewall_rules[count.index].destination
  
  # ... other configuration
}

output "first_rule_name" {
  value = var.firewall_rules[0].name
}
```

---

## Comparison Table

| Type | Structure | Access Method | Best Use Case | Loop Method |
|------|-----------|---------------|---------------|-------------|
| `map(string)` | Key-value strings | `var.map["key"]` | Tags, configs, environment settings | `for_each` |
| `list(string)` | Ordered strings | `var.list[index]` | Subnets, availability zones | `count` |
| `map(object(...))` | Map of objects | `var.map["key"].attr` | Multiple VMs, structured resources | `for_each` |
| `list(object(...))` | List of objects | `var.list[index].attr` | Firewall rules, ordered configs | `count` |

---

## Decision Matrix

### When to use `map` vs `list`

**Choose `map` when:**
- You need to reference items by a meaningful key
- Order doesn't matter
- You want to add/remove items without affecting others
- You need to use `for_each` for resource creation

**Choose `list` when:**
- Order matters (e.g., firewall rules with priorities)
- You need sequential access
- You want to use `count` for resource creation
- Items don't have natural unique identifiers

### When to use `string` vs `object`

**Choose `string` when:**
- Simple key-value pairs are sufficient
- You only need one piece of data per item

**Choose `object` when:**
- You need multiple related attributes
- Complex configuration is required
- You want type safety for structured data

---

## Best Practices

### 1. Use Descriptive Types
```hcl
# Good
variable "vm_configs" {
  type = map(object({
    vm_size    = string
    os_type    = string
    disk_size  = number
  }))
}

# Avoid
variable "configs" {
  type = map(any)
}
```

### 2. Provide Validation
```hcl
variable "environments" {
  type = list(string)
  validation {
    condition = alltrue([
      for env in var.environments : contains(["dev", "staging", "prod"], env)
    ])
    error_message = "Environments must be one of: dev, staging, prod."
  }
}
```

### 3. Use for_each with Maps
```hcl
# Preferred for maps
resource "azurerm_virtual_machine" "example" {
  for_each = var.vm_configs
  name     = each.key
  size     = each.value.vm_size
}
```

### 4. Use count with Lists
```hcl
# Preferred for lists where order matters
resource "azurerm_network_security_rule" "example" {
  count    = length(var.security_rules)
  name     = var.security_rules[count.index].name
  priority = var.security_rules[count.index].priority
}
```

---

## Rule of Thumb

- **`map(string)`** → Simple key-value lookup (tags, environment configs)
- **`list(string)`** → Ordered simple values (subnets, AZs)
- **`map(object)`** → Complex resources by name (VMs, databases)
- **`list(object)`** → Ordered complex configurations (firewall rules, policies)

Choose the type that matches your data structure and access patterns!
