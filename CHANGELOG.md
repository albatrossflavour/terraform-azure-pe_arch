# Changelog

## Unreleased

### Changed

- Upgraded `hashicorp/azurerm` from 2.64.0 to 5.7.0, `hashicorp/random` from 3.1.0 to 3.9.1 and `chriskuchin/hiera5` from 0.3.0 to 0.5.4.
- Raised `required_version` to `>= 1.5` for OpenTofu 1.x and Terraform 1.x.
- Changed the default `instance_image` from `almalinux:almalinux:8-gen2:latest` to `almalinux:almalinux-x86_64:9-gen2:latest` and the default `image_plan` to an empty string. The legacy `almalinux:almalinux` Marketplace offer was deprecated at the end of 2024 and its replacement does not use a plan.
- Load balancer probe protocol is now `Tcp`, as azurerm 3.0 made the value case sensitive.
- Removed `resource_group_name` from `azurerm_lb_probe` and `azurerm_lb_rule`, which azurerm 3.0 dropped in favour of `loadbalancer_id`.
- `azurerm_lb_rule` now uses `backend_address_pool_ids` in place of the removed `backend_address_pool_id`.
- The hiera5 provider is now pointed at `hiera.yaml` explicitly. From 0.4.0 it defaults to `hiera.yml`, and every lookup failed with "key not found" without this.
- The azurerm provider registers `Microsoft.Compute` and `Microsoft.Network`, as azurerm 5.0 stopped registering resource providers by default.
- The azurerm provider sets `prevent_deletion_if_contains_resources = false` to keep the pre 3.0 destroy behaviour for the resource group.
- Formatted all files with `tofu fmt`.

### Added

- Optional `subscription_id` variable, passed to the azurerm provider. azurerm 4.0 made the subscription ID mandatory. When the variable is null the provider uses `ARM_SUBSCRIPTION_ID` or the Azure CLI default subscription.

### Notes

- azurerm 4.0 changed the default SKU of `azurerm_public_ip` and `azurerm_lb` from Basic to Standard. The module does not set a SKU, so new deployments get Standard public IPs and a Standard load balancer. Basic SKUs can no longer be created in Azure.
- State written by azurerm 2.64.0 has not been tested against 5.7.0. Destroy old deployments with the old module version, or expect to plan carefully.
