# Haskell Dev Container

A ready-to-use **Haskell development container** for Visual Studio Code using Dev Containers.

The container provides a reproducible Haskell toolchain based on Debian Bookworm, with GHC, Cabal, GHCup, HLS, GHCi tools preinstalled.

## Requirements

You will need:

* [Visual Studio Code](https://code.visualstudio.com/)
* [Docker](https://www.docker.com/)
* VS Code's **Dev Containers** extension (`ms-vscode-remote.remote-containers`)


## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Ryan-Diazz/haskel-dev-container
cd haskel-dev-container
```

### 2. Open the repository in VS Code

```bash
code .
```

VS Code should detect the Dev Container configuration.

Alternatively, open the Command Palette:

```
Ctrl+Shift+P
```

and select:

```text
Dev Containers: Reopen in Container
```

VS Code will build the development image and open the repository inside the container. 

> [!NOTE]
> The build will take a few minutes, but after that it will open faster.


## Usage

### Project Structure
```text
/
├── .devcontainer/
│   ├── Dockerfile
│   └── devcontainer.json
├── src/
└── README.md
```

the `src/` folder is meant for the `.hs` files.

The exact project structure is not imposed by the container, but newer versions of the container could be pulled if everything is located in `src/` as it is ignored in `.gitignore`.


### Included Tools

#### GHC

The Glasgow Haskell Compiler is installed through GHCup.

```bash
ghc --version
```

#### GHCi

The interactive Haskell interpreter is available directly:

```bash
ghci
```

Example:

```haskell
ghci> 2 + 2
4
```

#### Cabal

Cabal is available for building and managing Haskell projects:

```bash
cabal --version
```

Create a new project with:

```bash
cabal init
```

Build a project with:

```bash
cabal build
```

Run the project's tests with:

```bash
cabal test
```

#### Haskell Language Server

HLS is installed and provides VS Code features such as:

* Code completion
* Diagnostics
* Go to definition
* Hover information
* Code actions
* Formatting
* Refactoring support

The VS Code Haskell extension can use HLS automatically.


The local Hoogle database is generated during the Docker image build.

## direnv

The container also includes [`direnv`](https://direnv.net/) and automatically enables its Bash and Zsh integration.

The following is added to the shell configuration:

```bash
eval "$(direnv hook bash)"
```

This allows projects to define environment variables using a `.envrc` file.

For example:

```bash
echo 'export MY_VARIABLE="hello"' > .envrc
direnv allow
```

Then:

```bash
echo "$MY_VARIABLE"
```

will output:

```text
hello
```

## Haskell Toolchain Versions

The container uses [GHCup](https://www.haskell.org/ghcup/) to install and manage the Haskell toolchain.

The default versions are configured as arguments in the `devcontainer.json`.
To use a different version, change these arguments and rebuild the container.

For example:

```json
"args": {
    // "GOLANG_VERSION": "1.26.3",
    // "DIRENV_VERSION": "2.37.1",
    "GHC_VERSION": "9.10.3",
    "CABAL_VERSION": "3.12.1.0"
}
```

After changing versions, rebuild using vscode command:

```text
Dev Containers: Rebuild Container
```

access using `Ctrl+Shift+P`


## Container User

Development takes place as the non-root user:

```text
vscode
```

with UID `1000`.

The default workspace is:

```text
/workspaces/haskell
```

This keeps files created inside the container compatible with a typical local Linux development environment.

## Building the Image

The image can also be built manually:

```bash
docker build -t haskell-dev .
```

The build accepts the following arguments:

| Argument         | Default    | Description                           |
| ---------------- | ---------- | ------------------------------------- |
| `GHC_VERSION`    | `9.10.3`   | GHC version installed through GHCup   |
| `CABAL_VERSION`  | `3.12.1.0` | Cabal version installed through GHCup |
| `GOLANG_VERSION` | `1.26.3`   | Go version used to build direnv       |
| `DIRENV_VERSION` | `2.37.1`   | direnv version                        |

For example:

```bash
docker build \
    --build-arg GHC_VERSION=9.10.3 \
    --build-arg CABAL_VERSION=3.12.1.0 \
    -t haskell-dev .
```

## Useful Commands

| Command         | Purpose                           |
| --------------- | --------------------------------- |
| `ghc --version` | Show GHC version                  |
| `ghci`          | Start the Haskell interpreter     |
| `cabal build`   | Build the project                 |
| `cabal run`     | Run the project                   |
| `cabal test`    | Run tests                         |
| `cabal repl`    | Start a project REPL              |
| `cabal clean`   | Remove build artifacts            |
| `ghcup list`    | List available toolchain versions |
| `ghcup tui`     | Open the GHCup interface          |
| `direnv status` | Show direnv status                |

## Customization

The image is intentionally kept relatively minimal while providing the tools needed for Haskell development.

Additional system dependencies can be added to the final image with:

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    <package> \
    && rm -rf /var/lib/apt/lists/*
```

Haskell packages can be added in the builder stage with the code block:

```dockerfile
RUN cabal install \
    <package> \
    --overwrite-policy=always
```

## Troubleshooting

### HLS is not working in VS Code

Verify that HLS is installed:

```bash
haskell-language-server-wrapper --version
```

Then make sure the VS Code Haskell extension is installed inside the Dev Container.

You can also inspect the VS Code Output panel and select the Haskell Language Server output.

### Changes to the Dockerfile are not appearing

VS Code may be using a previously built image.

Run:

```text
Dev Containers: Rebuild Container
```

or:

```text
Dev Containers: Rebuild Container Without Cache
```

if you need to force all installation steps to run again.