# bug: lorri displays "error extending watch paths" warning during operation

This repository is a minimal reproduction of a problem encountered on:

- macOS 14.4.1 Sonoma
- lorri 1.7.1 installed via `service.lorri.enable = true;`
- nixpkgs is `github:NixOS/nixpkgs/85f7e662eda4fa3a995556527c87b2524b691933?narHash=sha256-JwQZIGSYnRNOgDDoIgqKITrPVil%2BRMWHsZH1eE1VGN0%3D (2024-11-07 05:50:23)`


The `.envrc` and `shell.nix` files were created by running `lorri init`.

```shell
# uname -a
Darwin pld-mbp-22 23.4.0 Darwin Kernel Version 23.4.0: Fri Mar 15 00:10:42 PDT 2024; root:xnu-10063.101.17~1/RELEASE_ARM64_T6000 arm64 arm Darwin

# lorri --version
lorri 1.7.1

# ls -al $(which lorri)
lrwxr-xr-x 1 root wheel 65 Dec 31  1969 /run/current-system/sw/bin/lorri -> /nix/store/s0adm0g33af23fm8dr0vqnvvh2gmznrf-lorri-1.7.1/bin/lorri

# nix --version
nix (Nix) 2.24.10

# lorri info --shell-file shell.nix
Project Shell File: /Users/pd/Desktop/lorri-init-path-errors/shell.nix
Project Garbage Collector Root: /Users/pd/Library/Caches/com.github.nix-community.lorri.lorri.lorri/gc_roots/bad6431c93487fe2821f8b802ac8a0d8/gc_root/shell_gc_root

General:
Lorri User GC Root Dir: /Users/pd/Library/Caches/com.github.nix-community.lorri.lorri.lorri/gc_roots
Lorri Daemon Socket: /Users/pd/Library/Caches/com.github.nix-community.lorri.lorri.lorri/daemon.socket
Lorri Daemon Status: `lorri daemon` is running
```

After running `direnv allow .`, you can reproduce the issue by running:

```shell
# lorri watch --once
Nov 08 16:51:03.439 WARN error extending watch paths:, paths: [Normal("/Users/pd/.config/nixpkgs/config.nix"), Normal("<nix/fetchurl.nix>"), Normal("/Users/pd/Desktop/lorri-init-path-errors/shell.nix"), Normal("<nix/derivation-internal.nix>")], error: Error { kind: Io(Os { code: 2, kind: NotFound, message: "No such file or directory" }), paths: [] }, nix_file: /Users/pd/Desktop/lorri-init-path-errors/shell.nix
Nov 08 16:51:03.489 INFO build message, message: OutputPath { shell_gc_root: RootPath(AbsPathBuf("/Users/pd/Library/Caches/com.github.nix-community.lorri.lorri.lorri/gc_roots/bad6431c93487fe2821f8b802ac8a0d8/gc_root/shell_gc_root")) }, nix_file: /Users/pd/Desktop/lorri-init-path-errors/shell.nix
direnv: loading ~/Desktop/lorri-init-path-errors/.envrc
direnv: export +AR +AS +CC +CONFIG_SHELL +CXX +DEVELOPER_DIR +HOST_PATH +IN_LORRI_SHELL +IN_NIX_SHELL +LD +LD_DYLD_PATH +MACOSX_DEPLOYMENT_TARGET +NIX_APPLE_SDK_VERSION +NIX_BINTOOLS +NIX_BINTOOLS_WRAPPER_TARGET_HOST_aarch64_apple_darwin +NIX_BUILD_CORES +NIX_BUILD_TOP +NIX_CC +NIX_CC_WRAPPER_TARGET_HOST_aarch64_apple_darwin +NIX_CFLAGS_COMPILE +NIX_DONT_SET_RPATH +NIX_DONT_SET_RPATH_FOR_BUILD +NIX_DONT_SET_RPATH_FOR_TARGET +NIX_ENFORCE_NO_NATIVE +NIX_HARDENING_ENABLE +NIX_IGNORE_LD_THROUGH_GCC +NIX_LDFLAGS +NIX_LOG_FD +NIX_NO_SELF_RPATH +NIX_STORE +NM +OBJCOPY +OBJDUMP +PATH_LOCALE +RANLIB +SDKROOT +SIZE +SOURCE_DATE_EPOCH +STRINGS +STRIP +ZERO_AR_DATE +__darwinAllowLocalNetworking +__impureHostDeps +__propagatedImpureHostDeps +__propagatedSandboxProfile +__sandboxProfile +__structuredAttrs +allowSubstitutes +buildInputs +buildPhase +builder +cmakeFlags +configureFlags +depsBuildBuild +depsBuildBuildPropagated +depsBuildTarget +depsBuildTargetPropagated +depsHostHost +depsHostHostPropagated +depsTargetTarget +depsTargetTargetPropagated +doCheck +doInstallCheck +extraClosure +mesonFlags +name +nativeBuildInputs +origArgs +origBuilder +origExtraClosure +origOutputs +origPATH +origSystem +out +outputs +patches +phases +preHook +preferLocalBuild +propagatedBuildInputs +propagatedNativeBuildInputs +shell +shellHook +stdenv +strictDeps +system ~PATH ~XDG_DATA_DIRS
# hello
Hello, world!
```

The problem is the presence of the path-related warning. I do not know which file or directory it thinks is missing. Lorri still functions correctly and provides the `hello` binary in the shell, as expected.

I believe Lorri should not be showing this warning, since it seems to be functioning properly. Additionally, the warning could be improved to include information about which path it is unable to watch.
