# Devfile v2.3.0 Schema Reference

Full property reference for all devfile elements.

## Top-Level Properties

```yaml
schemaVersion: ""       # Required. Pattern: ^([2-9])\.([0-9]+)\.([0-9]+)(\-[0-9a-z-]+(\.[0-9a-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$
metadata: {}            # See Metadata section below
components: []          # See Components section below
commands: []            # See Commands section below
events: {}              # See Events section below
projects: []            # See Projects section below
starterProjects: []     # See Starter Projects section below
dependentProjects: []   # See Dependent Projects section below
parent: {}              # See Parent section below
variables: {}           # map<string, string> — key-value pairs for {{variable}} substitution
attributes: {}          # Free-form YAML attributes
```

## Metadata

```yaml
metadata:
  name: ""              # Used by tooling and registries as the primary identifier for the devfile.
  version: ""           # Version content (e.g. 1.0.0). Pattern: ^([2-9])\.([0-9]+)\.([0-9]+)(\-[0-9a-z-]+(\.[0-9a-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$ 
  displayName: ""       # Display name for UIs
  description: ""       # Human-readable description of the devfile.
  language: ""          # The primary programming language (e.g. "Java", "Python", "Go", "TypeScript", "C#")
  projectType: ""       # The type of project - typically the framework or platform. (e.g. "Node.js", "Spring Boot", "Quarkus", "Django", "Go")
  provider: ""          # Who provides or maintains the devfile (e.g. "Red Hat", "IBM", "Community")
  icon: ""              # Devfile icon URI or relative path
  website: ""           # Project's website URL (e.g. https://quarkus.io)
  supportUrl: ""        # Link to a page that provides support information
  tags: []              # Tags for categorization and filtering
  architectures: []     # Supported architectures. Values are restricted to an enum: amd64, arm64, ppc64le, s390x
  globalMemoryLimit: "" # Overall memory limit (e.g. "4Gi")
  attributes: {}        # Free-form YAML attributes. Deprecated - use top-level
```

## Components

### Container Component

A development container with an image, environment variables, resource limits, endpoints, and volume mounts

```yaml
- name: ""                      # Required. Max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  container:
    image: ""                   # Required. Container image.
    command: []                 # Overrides the container image's default entrypoint (e.g. ["node"])
    args: []                    # Arguments passed to the command (e.g. ["server.js", "--port", "3000"])
    memoryLimit: ""             # Container maximum memory (e.g. "2Gi")
    memoryRequest: ""           # Container minimum memory (e.g. "512Mi")
    cpuLimit: ""                # Container maximum CPU (e.g. "2")
    cpuRequest: ""              # Container minimum CPU (e.g. "500m")
    mountSources: bool          # Default: true, except for containers with dedicatedPod: true. Whether project sources are mounted into the container
    sourceMapping: ""           # Default: /projects. Where project sources are mounted    
    dedicatedPod: bool          # Default: false. Run the container in its own separate pod instead of sharing the main pod with other containers
    env:                        # A list of environment variables set in the container ($PROJECTS_ROOT, $PROJECT_SOURCE are reserved)
      - name: ""                # Required
        value: ""               # Required
    endpoints: []               # Exposed ports by the container. See Endpoints section below
    volumeMounts:               # References to volume components to mount into the container
      - name: ""                # Required. Max 63 chars. Must match a volume component name
        path: ""                # Default: /<name>. Mount path in container (e.g. /database)
    annotation:                 # Annotations for Kubernetes resources
      deployment: {}            # Annotations added to deployment
      service: {}               # Annotations added to service
  attributes: {}                # Free-form YAML attributes
```

### Kubernetes / OpenShift Component

Imports Kubernetes/OpenShift resources (Deployments, Services, etc.) from a URI or inline manifest.

```yaml
- name: ""                      # Required. Max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  kubernetes:                   # OR: openshift:
    uri: ""                     # URL or relative path to a K8s/OpenShift manifest (e.g. "deploy/deployment.yaml", "https://example.com/k8s/service.yaml")
    # OR
    inlined: ""                 # Inline Kubernetes manifest (e.g. a Deployment, Service, or Route YAML)
    
    deployByDefault: bool       # Default: false. Auto-deploy Kubernetes resource on devworkspace start
    endpoints: []               # Exposed ports by the Kubernetes or OpenShift resource. See Endpoints section below
  attributes: {}                # Free-form YAML attributes
```

### Image Component

A Dockerfile-based image build

```yaml
- name:  ""                     # Required. Max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  image:
    imageName: ""               # Required. Name of the resulting build image
    autoBuild: bool             # Default: false. Auto-build on devworkspace start
    dockerfile:                 # Required. Dockerfile-based image build. One of: uri, git, devfileRegistry
      uri: ""                   # URI or relative path to a Dockerfile (e.g. "Dockerfile", "https://example.com/Dockerfile")
      # OR
      git:
        remotes: {}             # Required. map<string, string> of remote names to git URLs
        checkoutFrom:           # Required if multiple remotes configured. Which remote and revision to check out from
          remote: ""            # Remote name to init from (e.g. origin)
          revision: ""          # Branch, tag, or commit ID (e.g. main)
        fileLocation: ""        # Default: Dockerfile. Path to Dockerfile in repository.
      # OR
      devfileRegistry:          # Dockerfile from registry
        id: ""                  # Required. ID in a devfile registry that contains a Dockerfile (e.g. python)
        registryUrl: ""         # Required. Registry URL (e.g. https://registry.devfile.io)
        
      buildContext: ""          # Default: ${PROJECT_SOURCE}. Path of source directory to establish build context
      args: []                  # Build arguments (e.g. ['KEY=value'])
      rootRequired: bool        # Default: false. Requires root privileges to build
  attributes: {}                # Free-form YAML attributes
```

### Volume Component

A shared persistent volume that can be mounted into container components

```yaml
- name: ""              # Required. Max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  volume:
    size: ""            # Volume size (e.g. "1Gi")
    ephemeral: bool     # Default: false. Not persisted across restarts    
  attributes: {}        # Free-form YAML attributes
```

#### Endpoints

```yaml
- name: ""                # Required, max 15 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  targetPort: number      # Required. Port number inside the container
  exposure: ""            # Default: public. Values are restricted to an enum: public, internal, none
  protocol: ""            # Default: http. Values are restricted to an enum: http, https, ws, wss, tcp, udp
  secure: bool            # Whether endpoint requires authentication. Requires https or wss protocol
  path: ""                # Path of the endpoint URL (e.g. /api)
  annotation: {}          # Annotations for Kubernetes Ingress or OpenShift Route
  attributes: {}          # Free-form YAML attributes
```

## Commands

### Exec Command

Runs a shell command in a container component.

```yaml
- id: ""                      # Required, max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  exec:
    component: ""             # Required. Name of the container component to run in (e.g. tooling)
    commandLine: ""           # Required. The command-line string to execute, supports special variables $PROJECTS_ROOT, $PROJECT_SOURCE (e.g. mvn clean install)
    workingDir: ""            # Working directory for the command, supports special variables $PROJECTS_ROOT, $PROJECT_SOURCE (e.g. /home/user)
    env:                      # Environment variables set before running the command
      - name: ""              # Required
        value: ""             # Required
    group:
      kind: ""                # Required. Values are restricted to an enum: build, run, test, debug, deploy
      isDefault: bool         # Default command for this group kind.
    hotReloadCapable: bool    # Default false. Specify whether the command is restarted or not when the source code changes
    label: ""                 # Human-readable label (e.g. Build a project)
  attributes: {}              # Free-form YAML attributes
```

### Apply Command

Applies a component definition, typically deploying Kubernetes/OpenShift resources or triggering an image build

```yaml
- id: ""                      # Required, max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  apply:
    component: ""             # The name of an existing kubernetes, openshift, or image component to apply (e.g. openshift-route)
    group:
      kind: ""                # Required. Values are restricted to an enum: build, run, test, debug, deploy
      isDefault: bool         # Default command for this group kind.
    label: ""                 # Human-readable label (e.g. Create an application route)
  attributes: {}              # Free-form YAML attributes
```

### Composite Command

Groups multiple exec and/or apply commands into a single command

```yaml
- id: ""                      # Required, max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  composite:
    commands: []              # List of existed command IDs to execute
    parallel: false           # Default: false. Run commands concurrently
    group:
      kind: ""                # Required. Values are restricted to an enum: build, run, test, debug, deploy
      isDefault: bool         # Default command for this group kind.
    label: ""                 # Human-readable label (e.g. Create an application route)
  attributes: {}              # Free-form YAML attributes
```

## Events

preStart, postStart, preStop, postStop lifecycle hooks

```yaml
events:
  preStart: []      # Commands executed before devworkspace starts, run as init containers
  postStart: []     # Commands executed after devworkspace is fully started
  preStop: []       # Commands executed before devworkspace stop
  postStop: []      # Commands executed after devworkspace stops
```

## Projects

Projects worked on in the devworkspace, containing names and sources locations

```yaml
- name: ""                  # Required, max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  git:
    remotes: {}             # Required. map<string, string> of remote names to git URLs
    checkoutFrom:           # Required if multiple remotes configured. Which remote and revision to check out from
      remote: ""            # Remote name to init from (e.g. origin)
      revision: ""          # Branch, tag, or commit ID (e.g. main)
  # OR
  zip:
    location: ""            # Zip project's source location address (e.g. https://host/project.zip)
    
  clonePath: ""             # Path relative to the root of the projects to which this project should be cloned into
  attributes: {}            # Free-form YAML attributes
```

## Starter Projects

A project that can be used as a starting point when bootstrapping new projects

```yaml
- name: ""                  # Required, max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  git:
    remotes: {}             # Required. map<string, string> of remote names to git URLs
    checkoutFrom:           # Required if multiple remotes configured. Which remote and revision to check out from
      remote: ""            # Remote name to init from (e.g. origin)
      revision: ""          # Branch, tag, or commit ID (e.g. main)
  # OR
  zip:
    location: ""            # Zip project's source location address (e.g. https://host/project.zip)
    
  description: ""           # Human-readable project description
  subDir: ""                # Sub-directory from a starter project to be used as root for starter project
  attributes: {}            # Free-form YAML attributes
```

## Dependent Projects

Additional projects related to the main project in the devfile, containing names and sources locations

```yaml
- name: ""                  # Required, max 63 chars, pattern: ^[a-z0-9]([-a-z0-9]*[a-z0-9])?$
  git:
    remotes: {}             # Required. map<string, string> of remote names to git URLs
    checkoutFrom:           # Required if multiple remotes configured. Which remote and revision to check out from
      remote: ""            # Remote name to init from (e.g. origin)
      revision: ""          # Branch, tag, or commit ID (e.g. main)
  # OR
  zip:
    location: ""            # Zip project's source location address (e.g. https://host/project.zip)
    
  clonePath: ""             # Path relative to the root of the projects to which this project should be cloned into
  attributes: {}            # Free-form YAML attributes
```

## Parents

Parent devworkspace template

```yaml
parent:
  registryUrl: ""             # Registry URL to pull the parent devfile from when using id in the parent reference (e.g. https://registry.devfile.io/)
  id: ""                      # Stack ID in registry (e.g. nodejs)
  version: ""                 # Version (e.g. 2.2.0, latest). Pattern: ^(latest)|([2-9])\.([0-9]+)\.([0-9]+)(\-[0-9a-z-]+(\.[0-9a-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$

  # OR
  uri: ""                     # URI Reference of a parent devfile YAML file. It can be a full URL or a relative URI with the current devfile as the base URI

  # OR
  kubernetes:
    name: ""                  # Required. Custom Resource name of type DevWorkspaceTemplate
    namespace: ""             # Custom Resource namespace

  attributes: {}                # Override parent attributes
  variables: {}                 # Override parent variables
  components: []                # Override parent components 
  commands: []                  # Override parent commands
  projects: []                  # Override parent projects
  dependentProjects: []         # Override parent dependent projects
  starterProjects: []           # Override parent starter projects
```
