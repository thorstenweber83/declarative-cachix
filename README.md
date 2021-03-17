# Declarative Cachix

Add [cachix](https://cachix.org/) caches declaratively.
You can use it either as a system module, or as a home-manager module.

### System module

Import `default.nix` into your system configuration.
This adds the top-level `cachix` option, which you can use to add cachix caches.
You can either pass them as names, or as `{name, sha256}` attribute pairs.

Example configuration:
```nix
  {
    imports = [
      (import (builtins.fetchTarball "https://github.com/jonascarpay/declarative-cachix/archive/a2aead56e21e81e3eda1dc58ac2d5e1dc4bf05d7.tar.gz"))
    ];

    cachix = [
      { name = "jmc"; sha256 = "1bk08lvxi41ppvry72y1b9fi7bb6qvsw6bn1ifzsn46s3j0idq0a"; }
      "iohk"
    ];
  }
```

### Home-manager module

#### Warning
Home-manager does not contain a mechanism for declaratively adding caches like the system config does.
This module adds that by generating a `.config/nix/nix.conf` file declaratively.

This has two important implications:
  1. If you already have entries in that file aside from the default nix cache, you need to move those to `home.file.nixConf.text` so they get included in the generated file.
  2. If the file is somehow malformed, it will break home-manager itself, so you then have to manually delete/fix it and fix your config. I haven't had any issues myself, but caveat emptor. Under most circumstances, you should be sufficient to remove the symlink and replace it with the default contents:
```
substituters = https://cache.nixos.org
trusted-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
```

#### Usage

Import `home-manager.nix` into your home-manager configuration.
This adds two user-facing options; `caches.extraCaches` and `caches.cachix`.
There's also `caches.caches`, but you typically don't want to set this manually since it also adds the normal nix hydra.

Note that you will need to be a trusted user.
See [this issue](https://github.com/jonascarpay/declarative-cachix/issues/2) for more information.

Example configuration:
```nix
  {
    imports = [
      (
        let
          declCachix = builtins.fetchTarball "https://github.com/jonascarpay/declarative-cachix/archive/a2aead56e21e81e3eda1dc58ac2d5e1dc4bf05d7.tar.gz";
        in import "${declCachix}/home-manager.nix"
      )
    ];

    caches.cachix = [
      { name = "jmc"; sha256 = "1bk08lvxi41ppvry72y1b9fi7bb6qvsw6bn1ifzsn46s3j0idq0a"; }
      "nix-community"
    ];

    caches.extraCaches = [
      {
        url = "https://hydra.iohk.io";
        key = "hydra.iohk.io:********************************************";
      }
    ];
  }
```
