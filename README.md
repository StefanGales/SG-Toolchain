# SG-Toolchain

The FPC and Lazarus installers that `StefanGales/SG-LPK` is built with, kept in one
place so that a build is always done with **the same versions Stefan builds with**.

That is the whole point. The `sg_p` package has produced exactly `31 / 687` warnings
and hints on Debian and `36 / 687` on Windows in every log since `SG_V5`, and that
pair is used as a check: a log that says 32 or 688 is telling you something about the
code just written. A CI job on "whatever Lazarus is current" would make the number
meaningless. So the versions are pinned here, by being *these files*.

Nothing in this repository is ours. FPC and Lazarus are (L)GPL and freely
redistributable; these are the unmodified official installers.

## What to attach, and where

Create **one release**, tagged exactly `toolchain`, and attach these files as release
assets — not as files committed to git:

| file | needed by | why |
|---|---|---|
| `fpc-laz_3.2.2-210709_amd64.deb` | Linux | the compiler and the RTL/FCL units |
| `fpc-src_3.2.2-210709_amd64.deb` | Linux | the Lazarus deb depends on it; a build does not read it, but the install fails without it |
| `lazarus-project_4.8.0-0_amd64.deb` | Linux | `lazbuild`, and the LCL and LazUtils units prebuilt for GTK2 |
| `lazarus-*-win64.exe` | Windows | the official Windows installer, whatever its exact name is |

**Release assets, not committed files.** GitHub refuses any file over 100 MB on a
push, and the Lazarus deb is close to or over that; a release asset may be up to 2 GB
and never enters the git history, so cloning this repository stays instant.

The workflow matches the assets by pattern (`fpc-laz_*_amd64.deb`,
`lazarus-project_*_amd64.deb`, `lazarus-*win64.exe`), so the exact version numbers in
the names do not have to be edited anywhere when the toolchain is updated. Replacing
the assets in the `toolchain` release is the whole of an upgrade.

## The token, because this repository is private

`SG-LPK`'s workflow cannot read a private repository with its own `GITHUB_TOKEN` —
that token is scoped to `SG-LPK` alone. One of the two:

* **keep it private** and give `SG-LPK` a token: Settings → Developer settings →
  Personal access tokens → Fine-grained tokens → new token, repository access
  *only* `SG-Toolchain`, permission **Contents: Read-only**. Then in `SG-LPK`:
  Settings → Secrets and variables → Actions → new repository secret named
  `TOOLCHAIN_TOKEN`.
* **or make this repository public**, and the workflow needs no secret at all. There
  is nothing private in it — it is four public installers.

The workflow fails with a message naming this file if the secret is missing, rather
than failing obscurely.

## The second consumer: the assistant's sandbox

The sandbox `SG-LPK` is worked on from has **no general network**: plain HTTPS to
github.com, SourceForge and the Debian mirrors is all refused. The one thing that
does work is `git clone` of a GitHub repository.

So if a compiler is ever wanted *inside* the sandbox — to catch a
`Duplicate identifier` in seconds instead of a round trip — the bytes have to arrive
through git, which means **committed to this repository**, not as a release asset.
That would be a second, separate thing to set up:

* unpack the two Linux debs once, keep only the compiler plus
  `lcl/units/x86_64-linux/{,gtk2}` and `components/lazutils/lib/x86_64-linux`, and
  commit that pruned tree — roughly 60–90 MB instead of 180;
* it would give Linux only, and two of the three most expensive failures in this
  project so far were Windows-only, which is why CI came first.

Not set up yet, and worth doing only if the CI round trip turns out to be too slow
in practice.
