+++
title = "IncludingWindowsDefaultRecipe"

+++

<!-- This content is automatically generated. See https://github.com/chef/chef-web-docs/blob/main/generated/README.md -->

The department is: `IncludingWindowsDefaultRecipe`

The full name of the cop is: `Chef/Modernize/IncludingWindowsDefaultRecipe`

| Enabled by default | Supports autocorrection | Target Chef Version |
| --- | --- | --- |
| Enabled | Yes | All Versions |

do not include the windows default recipe that is either full of gem install that are part of the Chef Infra Client, or empty (depends on version).

## Examples


#### incorrect

```ruby
include_recipe 'windows::default'
include_recipe 'windows'
```

## Configurable attributes

<table>
<tbody><tr>
<th>Name</th>
<th>Default value</th>
<th>Configurable values</th>
</tr>
<tr>
<td style="text-align:center">Version Added</td>
<td style="text-align:center">5.3.0</td>
<td style="text-align:center">String</td>
</tr>
<tr><td style="text-align:center">Include</td>
<td style="text-align:center"><ul>
</ul>
</td>
<td style="text-align:center">Array</td>
</tr></tbody></table>
