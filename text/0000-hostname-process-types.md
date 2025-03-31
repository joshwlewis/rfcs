# Meta

- Name: Hostname Compatible Process Types
- Start Date: 2025-03-30
- Author(s): @joshwlewis
- Status: Draft <!-- Acceptable values: Draft, Approved, On Hold, Superseded -->
- RFC Pull Request:
- CNB Pull Request:
- CNB Issue:
- Supersedes:

# Summary

Process types in the Cloud Native Buildpacks Buildpack specification are less restrictive than the RFC 1123 2.1 host name definition. Therefore, CNB process types may not be compatible with internet routing or kubernetes resource naming. This RFC proposes to restrict CNB process types to meet RFC 1123 2.1 and increase usage compatibility with other CNCF / OSS usage.

# Definitions

RFC 1123 Requirements for Internet Hosts Section 2.1: The IETF ["Requirements for Internet Hosts" RFC](https://datatracker.ietf.org/doc/html/rfc1123#section-2.1) describes the syntax for host names on the internet.
Kubernetes Resource Names: Many resource names are restricted to RFC 1123, as outlined [here](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names).
Process Types: The name of a process in a Cloud Native Buildpacks result as [described in `launch.toml`](https://github.com/buildpacks/spec/blob/main/buildpack.md#launchtoml-toml).

# Motivation

The images resulting from CNB builds are often deployed to internet connected environments, where it's useful to connect or manage the image lifecycle with a domain name.

For example, after building a web service image with Cloud Native Buildpacks, developers might want to expose various processes to the internet like static-assets.myapp.com and api.myapp.com. However, process types may currently include `_`, `.`, and/or `[A-Z]`, making process types like `Static_Assets` allowed, but not internet routable -- `Static_Assets.myapp.com` is not a valid domain name.

Or, developers might want to manage Kubernetes resources dynamically using process types defined in the resulting image:

```
kubectl autoscale replicaset static-assets --max=6 --min=2 --cpu-percent=50
```

However, while CNB allows a `Static_Assets` process type, Kubernetes rejects it as a resource name:

```
object name does not conform to Kubernetes naming requirements: "Static_Assets"
```

These incompatibilities make using process types in dynamic scenarios challenging and/or impossible.

# What it is

The current Buildpack specification defines these process type restrictions:

> MUST only contain numbers, letters, and the characters ., \_, and -.

While RFC 1123 uses a stricter syntax with these notable differences:

- Underscores (`_`) are not allowed
- Periods (`.`) are not allowed
- Dashes ('-') are not allowed as the first or last character
- Uppercase and lowercase letters are not differentiated
- Must be 63 characters or less

The buildpack specification will be modified to match RFC 1123 2.1.

# How it Works

The platform specification will be changed to reject process types that do not comply with RFC 1123 2.1. For maximum compatibility with Kubernetes resource names, the Kubernetes regular expression will be used:

```
/[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*/
```

When a process type fails to meet this expression, appropriate error messaging may be given by the platform, for example:

```
The "Static_Assets" process type must comply with RFC 1123 host name syntax. Underscores are not allowed. Uppercase letters are not allowed."
```

While uppercase letters are allowed in RFC 1123, uppercase and lowercase are not differentiated -- `API` and `api` are the same. DNS providers and Kubernetes enforce (or coerce) uppercase to lowercase to prevent duplicate entries with different casing. The buildpack spec should also enforce lowercase letters for the same reason.

# Migration

This is a breaking API change, and will require a new Buildpack API version.

This may require buildpacks to change the way process types are defined. For buildpacks using statically chosen process types that do not comply with RFC 1123, they'll need to change the chosen process types. For buildpacks using dynamically assigned process types, they'll need to coerce process types (by substitution and/or truncation) to meet the specification or risk builds failing when the dynamically chosen name does not comply.

# Drawbacks

- It's a breaking API change and will cause ecosystem churn.

# Alternatives

## Do Nothing

If we do nothing, developers in our ecosystem will not be able to use process types for internet routing or kubernetes resource names.

# Prior Art

N/A

# Unresolved Questions

- Are there additional resources in the spec that should also have this restriction?
  - Image names? (probably not, given it's common to use url-based image names with periods and slashes)

# Spec. Changes (OPTIONAL)

This change will require changes to the Buildpack API spec.

# History

<!--
## Amended
### Meta
[meta-1]: #meta-1
- Name: (fill in the amendment name: Variable Rename)
- Start Date: (fill in today's date: YYYY-MM-DD)
- Author(s): (Github usernames)
- Amendment Pull Request: (leave blank)

### Summary

A brief description of the changes.

### Motivation

Why was this amendment necessary?
--->

```

```
