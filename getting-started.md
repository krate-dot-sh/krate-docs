# Getting started with krate

krate runs each workload in its own microVM with its own kernel, and enforces
what that workload may do. Containers and devcontainers run unchanged, and no
Docker daemon is involved.

This page takes you from nothing to a running workload, then to your own project.

## Before you start

krate is in beta and currently runs on **macOS on Apple Silicon** (M1 or later).
It uses Apple's Virtualization framework to boot microVMs, which is why there is
no Intel or Linux build yet.

You do **not** need Docker installed.

## 1. Install

```bash
brew install krate-dot-sh/tap/krate
```

Check it landed:

```bash
krate info
```

`krate info` is the command to reach for whenever something looks wrong. It
reports the CLI version and build, which VM image is in use, whether you are
logged in, and which registry you are pointed at.

## 2. Log in

```bash
krate login
```

This opens a browser for GitHub or Google sign-in and stores a token under
`~/.krate/`. A krate account is what grants your machine a seat and lets it pull
images from the krate registry.

## 3. Run something

```bash
krate run hello-krate
```

That pulls a small image and boots it in its own microVM. You should see it
print and exit within a second or so.

What happened: krate fetched the image, created a VM with its own kernel, ran
the workload inside it, and tore the VM down when the process exited. Nothing
ran on your host kernel, and nothing was left behind.

### Look at what is running

```bash
krate ps                  # list workloads
krate logs <name>         # stream a workload's stdout
krate exec <name> -- sh   # get a shell inside one
krate stop <name>         # stop it
```

## 4. Run your own project

If your project already has a devcontainer or a compose file, there is nothing
to convert:

```bash
cd ~/your-project
krate up
```

`krate up` looks for a config in the current directory and uses the first one it
finds, in this order:

1. `docker-compose.yml` (or `.yaml`, `compose.yml`, `compose.yaml`)
2. `krate.toml`
3. `.devcontainer/devcontainer.json`

Use `--file` to name one explicitly. When you are done:

```bash
krate down
```

## Workload types

`krate run` defaults to `container`, but the same command runs four kinds of
workload, each in its own microVM:

| Type | What it is |
|---|---|
| `container` | an OCI image, the default |
| `process` | a binary in a minimal Linux root filesystem |
| `wasm` | a WebAssembly module under the WASI capability model |
| `unikernel` | boots directly on the hypervisor, with no OS kernel at all |

```bash
krate run --type wasm my-module
```

They are listed roughly from most compatible to most restrictive. A container
gives you a full Linux userland; a unikernel gives you the smallest attack
surface.

## Policy

Isolation decides what a workload *can* reach. Policy decides what it is
*allowed* to do, and krate enforces it in the daemon rather than asking the
workload to behave.

Scaffold one and look at what it generates:

```bash
krate policy init     # create a krate.policy file
krate policy show     # the merged effective policy for this project
krate policy lint     # check it before you rely on it
```

Policy is evaluated when a workload starts. On the Team tier an org's policy
syncs to every machine, so a rule you set centrally applies everywhere without
anyone opting in.

## Where to go next

- `krate <command> --help` works on every command and is the most current
  reference, since it comes from the binary you have installed.
- `krate info` whenever behavior surprises you. Most confusing situations turn
  out to be a stale daemon or being logged out.
- `krate werks` is a separate product for spec-driven agent builds. It runs on
  krate and is documented separately.

## Troubleshooting

**`krate run` hangs or fails right after install.** Run `krate info` and check
the VM image line. The first run may need to fetch the microVM artifact, which
is larger than the CLI.

**"not logged in".** Run `krate login`. Tokens live in `~/.krate/token.json`;
removing that file logs you out on that machine.

**Something works in one command and not another.** Check that the CLI and the
running daemon are the same build. `krate info` shows the CLI build, and `krate
run` warns when the daemon it is talking to was built from different source.
That mismatch is the single most common cause of confusing behavior.
