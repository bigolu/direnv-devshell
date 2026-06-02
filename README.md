# direnv-devshell

A direnv plugin for loading a Nix [numtide/devshell][devshell].

## Features

- Caching: The devshell is cached so reentering the environment is much faster. The cache is invalidated whenever a watched file changes.

- Fallback: If the plugin fails to load the devshell, it can fall back to the last cached devshell. You can enable this by setting the environment variable `DEVSHELL_DIRENV_FALLBACK` to `true`.

## Differences from [nix-direnv][nix-direnv]

- This plugin only supports [numtide/devshell][devshell].

- No manual reload: I don't see the need for it.

- [nix-direnv][nix-direnv] will only fall back to the old environment if the _build_ of
  the new environment fails. This plugin will also fall back if the _sourcing_ of the
  new environment fails. _Sourcing_ refers to running the bash script from the
  devshell that creates the environment (setting `PATH`, etc.). This is the script
  that you can add to using the `devshell.startup` option.

- No GC root creation: Devshells already provide a way to set up the
  environment using the `devshell.startup` option. Therefore, they should be
  able to handle environment management themselves and not have to depend on
  a tool like direnv for creating a GC root. To help with this, I wrote a devshell
  module that creates a GC root. This module, and others, can be found in the
  [devshell-modules][devshell-modules] repository.

- No automatic file watching: I don't think there's a way to do it that will
  satisfy all use cases so users will still have to run `watch_{file,dir}`
  themselves. It's also a bit expensive to call since it runs `direnv` so
  ideally you only call it once. Instead, you can try something like the
  following which should work for most cases:

  ```bash
  shopt -s globstar
  watch_file **/*.nix **/flake.lock
  shopt +s globstar
  ```

## Usage

`use_devshell <build_arguments>...`

### `build_arguments`

These get appended to the command `nix build` to build the devshell. For example, if your `.envrc` contains `use_devshell my args` then this plugin will run the command `nix build my args` to build the devshell.

### Example for flake users

```bash
source_url \
  'https://raw.githubusercontent.com/bigolu/direnv-devshell/0f1a452b46c183c565b6ac5f710e9ea24bf918bb/src/main.bash' \
  'sha256-heBPLeA31k9eQvGJBKlxxkuMQXYGOOd4qyceg8aBBYQ='
DEVSHELL_DIRENV_FALLBACK=true use_devshell ".#devShells.$(uname -m)-$(uname -s | tr '[:upper:]' '[:lower:]').default"
```

[devshell]: https://github.com/numtide/devshell
[nix-direnv]: https://github.com/nix-community/nix-direnv
[devshell-modules]: https://github.com/bigolu/devshell-modules
