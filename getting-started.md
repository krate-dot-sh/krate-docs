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

If your project already has a compose file or a devcontainer, there is nothing
to convert. Here is the whole thing end to end.

Make a directory with a `compose.yml`:

```yaml
services:
  cache:
    image: redis:7-alpine
```

Then start it:

```bash
krate up
```

```
detected compose file: /path/to/your-project/compose.yml
Started (1 services)
  ✓ cache (192.168.64.40)
```

Each service gets its own microVM and its own IP on the krate network. Check it:

```bash
krate ps
```

```
ID            NAME             IP                PORTS           STATUS
------------------------------------------------------------------------
wl-000001     env-0001/cache   192.168.64.40     -               > Running
```

`krate up` looks for a config in the current directory and uses the first one it
finds, in this order:

1. `docker-compose.yml` (or `.yaml`, `compose.yml`, `compose.yaml`)
2. `krate.toml`
3. `.devcontainer/devcontainer.json`

Use `--file` to name one explicitly.

### Stopping it

```bash
krate down            # stop the environment for this directory
krate env destroy     # stop it and remove all state and caches
```

`krate env stop` does the same as `krate down`, and also takes an environment
id so you can stop one from anywhere rather than from its directory.

## Where to go next

You have krate installed, a workload running, and your own project booting.
Those are the next things worth learning, each its own topic:

- **Workload types.** `krate run` defaults to a container, but the same command
  runs processes, WebAssembly modules and unikernels, each in its own microVM
  and each with a smaller attack surface than the last.
- **Policy.** Isolation decides what a workload *can* reach. Policy decides what
  it is *allowed* to do, enforced in the daemon rather than requested of the
  workload. Start with `krate policy init`.
- **werks.** A separate product for spec-driven agent builds that runs on krate.

`krate <command> --help` works on every command and is the most current
reference, since it comes from the binary you have installed.

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
