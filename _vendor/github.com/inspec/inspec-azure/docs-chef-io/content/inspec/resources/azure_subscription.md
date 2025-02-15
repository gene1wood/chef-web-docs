+++
title = "azure_subscription Resource"
platform = "azure"
draft = false
gh_repo = "inspec-azure"

[menu.inspec]
title = "azure_subscription"
identifier = "inspec/resources/azure/azure_subscription Resource"
parent = "inspec/resources/azure"
+++

Use the `azure_subscription` InSpec audit resource to test properties of the current subscription.

## Azure REST API Version, Endpoint, and HTTP Client Parameters

{{% inspec_azure_common_parameters %}}

## Installation

{{% inspec_azure_install %}}

## Syntax

This resource will retrieve the current subscription id that InSpec uses unless it is provided via `id` or `resource_id` parameters.
```ruby
describe azure_subscription do
  it { should exist }
end
```
or
```ruby
describe azure_subscription(id: '2e0b423p-aaaa-bbbb-1111-ee558463aabbd') do
  it { should exist }
end
```
or
```ruby
describe azure_subscription(resource_id: '/subscriptions/2e0b423p-aaaa-bbbb-1111-ee558463aabbd') do
  it { should exist }
end
```

## Parameters

`id`
: The ID of the target subscription. `2e0b423p-aaaa-bbbb-1111-ee558463aabbd`.

`resource_id`
: The fully qualified ID for the subscription. `/subscriptions/2e0b423p-aaaa-bbbb-1111-ee558463aabbd`.

`locations_api_version`
: The endpoint api version for the `locations` property. Optional. The latest version will be used unless provided.

## Properties

`name`
: The subscription display name.

`id`
: The subscription ID. `2e0b423p-aaaa-bbbb-1111-ee558463aabbd`.

`locations`
: The list of all available geo-location names that have the `metadata.physicalLocation` is set.

`all_locations`
: The list of all available geo-location names. This includes physical and logical locations.

`physical_locations<superscript>*</superscript>`
: The list of all available geo-location names that have the `metadata.regionType` is set to `Physical`.

`logical_locations`
: The list of all available geo-location names that have the `metadata.regionType` is set to `Logical`.

`locations_list`
: The list of all available geo-location objects in [this](https://docs.microsoft.com/en-us/rest/api/resources/subscriptions/listlocations#location) format.

`managedByTenants`
: An array containing the [tenants](https://docs.microsoft.com/en-us/rest/api/resources/subscriptions/get#managedbytenant) managing the subscription.

`diagnostic_settings`
: The diagnostic settings set at a subcription level.

`diagnostic_settings_enabled_logging`
: The enabled logging types from diagnostic settings set at a subcription level.

`diagnostic_settings_disabled_logging`
: The disabled logging types from diagnostic settings set at a subcription level.

<superscript>*</superscript> `physical_locations` might be different than the `locations` property depending on the api version.
This is because of the change in the Azure API terminology. It is advised to see the [official documentation](https://docs.microsoft.com/en-us/rest/api/resources/subscriptions/listlocations) for more info.

For properties applicable to all resources, such as `type`, `properties`, refer to [`azure_generic_resource`]({{< relref "azure_generic_resource.md#properties" >}}).

Also, refer to [Azure documentation](https://docs.microsoft.com/en-us/rest/api/resources/subscriptions/get#subscription) for other properties available. 
Any attribute in the response may be accessed with the key names separated by dots (`.`), eg. `properties.<attribute>`.

## Examples

**Test Your Subscription`s Display Name.**

```ruby
describe azure_subscription do
  its('name') { should cmp 'Demo Resources' }
end
```
**Test Your Subscription`s Authorization Source.**

```ruby
describe azure_subscription do
  its('authorizationSource') { should cmp 'RoleBased' }
end
```
**Test Your Subscription`s Locations.**

```ruby
describe azure_subscription do
  its('locations') { should include('eastus') }
end
```    
**Test Your Subscription`s enabled logging types (via diagnostic settings).**

```ruby
describe azure_subscription do
  its('diagnostic_settings_enabled_logging_types') { should include('ResourceHealth') }
end
```  
**Test Your Subscription`s disabled logging types (via diagnostic settings).**

```ruby
describe azure_subscription do
  its('diagnostic_settings_disabled_logging_types') { should include('Recommendation') }
end
```  

## Matchers

This InSpec audit resource has the following special matchers. For a full list of available matchers, please visit our [Universal Matchers page](https://www.inspec.io/docs/reference/matchers/).

### exists

```ruby
# If we expect a resource to always exist

describe azure_subscription do
  it { should exist }
end
# If we expect a resource to never exist

describe azure_subscription(id: 'fake_id') do
  it { should_not exist }
end
```

## Azure Permissions

{{% azure_permissions_service_principal role="contributor" %}}
