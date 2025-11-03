---
title: Install system wide rust toolchain through rust-overlay and proper setup nixvim
date: '2025-11-01'
categories:
    - post
tags:
    - rust 
    - nix
draft: true
---

## Problem

For most rust project I have, the system wide installion of rust is enough. 
I didn't bother to have a project specific `nix.flake` for it.
I use nixvim for my neovim configuration, the `rust-analyzer` was originally brought by nixvim.
However, I get problem when I start working with `proc-macro`. 
It complain to me:

For most of my Rust projects, a system-wide Rust installation is sufficient, so I haven't bothered creating a project-specific `nix.flake`.

I use nixvim for my Neovim configuration with some dummy lsp config (the source of the evil), which originally provided `rust-analyzer`. 
However, I ran into a problem when working with proc-macros. I got the following error:

```console
Diagnostics: 1. Failed to run proc-macro server from path /nix/store/p4xxxxxxxxxxxxxx-rust-default-1.89.0/libexec/rust-analyzer-proc-macro-srv, 
error: Custom { kind: Other, error: "The version of the proc-macro server (6) in your Rust toolchain is newer than the version supported by your rust-analyzer (5).
This will prevent proc-macro expansion from working. Please consider updating your rust-analyzer to ensure compatibility with your current toolchain." } [macro-error]
```

I installed Rust system-wide through the `rust-overlay`, pinning the version to `1.89.0`. 
Here’s the relevant snippet from my configuration.nix:

```nix
rust-bin.stable."1.89.0".default
```

### Investigation

- `rust-analyzer` is from nixvim config.
- 


## Solution

The solution is by explicitly set and tell `rustaceanvim` where to find the `rust-analyzer` binary.

Install the `rust-analyzer` from rust-overlay as well by putting it in the `configuration.nix`.

```console
rust-bin.stable."1.89.0".rust-analyzer
```

Then it is important in the nixvim's `rustaceanvim` config disable rust-analyzer in `lsp.servers`.
This is where the `rust-analyzer` being silently installed.

```nix
plugins = {
  lsp = {
    enable = true;
    servers = {
      rust_analyzer.enable = false;
      # Common language servers
      ...
    };
  };
  rustaceanvim = {
    enable = true;
    settings = {
      server = {
        # use system wide rust-analyzer
        cmd = [ "/run/current-system/sw/bin/rust-analyzer" ];
        default_settings = {
          rust-analyzer = {
              ...
          };
        };
      };
    };
  };
}
```

## Summary

Commands used cheatsheet

- nix check the dependent ref in store
- LspInfo
- commands to confirm the rust-analyzer is from nixvim.

Tips on working with nix

- restart shell (double check this is the real problem).
