---
layout: "docs"
page_title: "Version Constraints - Configuration Language"
---

# Version Constraints

-> **Note:** This page is about Terraform 0.12 and later. For Terraform 0.11 and
earlier, see
[0.11 Configuration Language](../configuration-0-11/index.html).


Anywhere that Terraform lets you specify a range of acceptable versions for
something, it expects a specially formatted string known as a version
constraint. Version constraints are used when configuring:

- [Modules](./modules.html)
- [Provider sources](./provider-sources.html)
- [The `required_version` setting](./terraform.html#specifying-a-required-terraform-version) in the `terraform` block.

## Version Constraint Syntax

Terraform's syntax for version constraints is very similar to the syntax used by
dependency management systems like Bundler and NPM.

```hcl
version = ">= 1.2.0, < 2.0.0"
```

A version constraint is a [string](./expressions.html#string-literals)
containing one or more conditions, which are separated by commas.

Each condition consists of an operator and a version number.

Version numbers should be a series of numbers separated by periods (like
`1.2.0`), optionally with a suffix to indicate a beta release.

The following operators are valid:

- `=` (or no operator): Allows only one exact version number. Cannot be combined
  with other conditions.

- `!=`: Excludes an exact version number.

- `>`, `>=`, `<`, `<=`: Comparisons against a specified version, allowing
  versions for which the comparison is true. "Greater-than" requests newer
  versions, and "less-than" requests older versions.

- `~>`: Allows the specified version, plus newer versions that only
  increase the _most specific_ segment of the specified version number. For
  example, `~> 0.9` is equivalent to `>= 0.9, < 1.0`, and `~> 0.8.4`, is
  equivalent to `>= 0.8.4, < 0.9`. This is usually called the pessimistic
  constraint operator.

## Version Constraint Behavior

A version number that meets every applicable constraint is considered acceptable.

Terraform consults version constraints to determine whether it has acceptable
versions of itself, any required provider plugins, and any required modules. For
plugins and modules, it will use the newest installed version that meets the
applicable constraints.

If Terraform doesn't have an acceptable version of a required plugin or module,
it will attempt to download the newest version that meets the applicable
constraints.

If Terraform isn't able to obtain acceptable versions of external dependencies,
or if it doesn't have an acceptable version of itself, it won't proceed with any
plans, applies, or state manipulation actions.

Both the root module and any child module can constrain the acceptable versions
of Terraform and any providers they use. Terraform considers these constraints
equal, and will only proceed if all of them can be met.

## Best Practices

### Module Versions

- When depending on third-party modules, require specific versions to ensure
  that updates only happen when convenient to you.

- For modules maintained within your organization, specifying version ranges
  may be appropriate if semantic versioning is used consistently or if there is
  a well-defined release process that avoids unwanted updates.

### Terraform Core and Provider Versions

- Reusable modules should constrain only their minimum allowed versions of
  Terraform and providers, such as `>= 0.12.0`. This helps avoid known
  incompatibilities, while allowing the user of the module flexibility to
  upgrade to newer versions of Terraform without altering the module.

- Root modules should use a `~>` constraint to set both a lower and upper bound
  on versions for each provider they depend on.