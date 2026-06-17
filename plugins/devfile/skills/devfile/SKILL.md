---
name: devfile
description: Use when creating, modifying, or reviewing devfile/devfile.yaml files or devworkspace configurations (devfile.io v2.3.0)
---

## Overview

A devfile is a YAML file defining containerized development environments — components, commands, events, and deployment flows.
This skill covers the devfile v2.3.0 schema.

**Schema reference:** Read `devfile-schema-reference.md` when you need exact property names, types, defaults, or constraints.

**Output format:** Always output the full devfile YAML in a fenced code block. Add a brief explanation only for non-obvious choices.

## Default Devfile

When starting from scratch, use this as the base:

```yaml
schemaVersion: 2.3.0
metadata:
  name: devfile
components:
  - name: tools
    container:
      image: quay.io/devfile/universal-developer-image:latest
```

## Language/Framework Requests

When asked about a devfile for a specific programming language or framework:

1. Fetch the registry index via `WebFetch` on `https://registry.devfile.io/index` — returns a JSON array of all stacks with `name`, `language`, `projectType`, `tags`, and `description` fields.
2. Pick the best match by language, framework, or tags.
3. Fetch the devfile via `WebFetch` on `https://registry.devfile.io/devfiles/{name}` — returns the raw devfile YAML.
4. Adapt the devfile to the user's needs (adjust image, add components, set resources).
5. Link to `https://registry.devfile.io/devfiles/{name}` as the source.

If no registry match exists, build the devfile from scratch using the default template and appropriate container image.

## Guidelines

### Container Component

- Include `mountSources: true` by default. Omit it only when the container uses `dedicatedPod: true`.
- For non-UDI images (anything other than `quay.io/devfile/universal-developer-image`), include `command: ['sleep', 'infinity']` to keep the container alive for exec commands.

### Volume Component

When adding a volume mount to a container, always create a matching volume component with the same name and size.
For ephemeral volumes, add `ephemeral: true`.

### Image Component

When adding an image component, add `buildContext: .`.

### Variable Substitution Rules

**Syntax:** `{{variable-name}}`

**Cannot substitute in:**
- `schemaVersion`, `metadata`, `parent` source
- Element identifiers: `command.id`, `component.name`, `endpoint.name`, `project.name`
- References to identifiers: event bindings, `exec.component`, `volumeMounts.name`
- String enums: `group.kind`, `endpoint.exposure`

**Undefined variables:** Non-blocking warning (devfile still processed).

### Reserved Environment Variables

These cannot be overridden via container `env`:
- `$PROJECTS_ROOT` — path where project sources are mounted (default `/projects`)
- `$PROJECT_SOURCE` — path to the default project source (`$PROJECTS_ROOT/<project-name>`)

### Pod & Container Overrides

```yaml
# Top-level pod overrides
attributes:
  pod-overrides:
    spec:
      __KUBERNETES_POD_SPEC_FIELDS_HERE__

# Component-level overrides
components:
  - name: tools
    attributes:
      container-overrides:
        __KUBERNETES_CONTAINER_FIELDS_HERE__
      pod-overrides:
        spec:
          __KUBERNETES_POD_SPEC_FIELDS_HERE__
```

Merge strategy: devfile-level `pod-overrides` are applied first, then component-level, using Strategic Merge Patch.

| Override Type | Restricted Properties |
|--------------|----------------------|
| `container-overrides` | `image`, `name`, `ports`, `env`, `volumeMounts`, `command`, `args` |
| `pod-overrides` | `containers`, `initContainers`, `volumes` |
