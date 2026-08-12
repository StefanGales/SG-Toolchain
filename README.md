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

## What is here now: the four installers, in Git LFS

| file | size | needed by |
|---|---|---|
| `fpc-laz_3.2.2-210709_amd64.deb` | 37.5 MB | Linux — the compiler and the RTL/FCL units |
| `fpc-src_3.2.2-210709_amd64.deb` | 29.2 MB | Linux — the Lazarus deb depends on it |
| `lazarus-project_4.8.0-0_amd64.deb` | 149.7 MB | Linux — `lazbuild`, and the LCL prebuilt for GTK2 |
| `lazarus-4.8-fpc-3.2.2-win64.exe` | 211.8 MB | Windows — the official installer |

They are **committed with Git LFS**, so what git holds is a 133-byte pointer per
file and the bytes live in GitHub's LFS storage. `SG-LPK`'s workflow handles this:
it clones this repository with `GIT_LFS_SKIP_SMUDGE=1` and then pulls only the files
the job in hand needs.

### Worth knowing: this costs LFS bandwidth on every run

A Linux run pulls 216 MB and a Windows run 212 MB, and **that comes off the
account's Git LFS bandwidth allowance** — 1 GB a month on a free account. So about
four runs a month before LFS starts refusing, which shows up as a failed build for a
reason that has nothing to do with the code.

**Attaching the same four files to a release instead has no such limit**, and the
workflow already prefers a release if it finds one: it looks for a release tagged
`toolchain` first and only falls back to LFS. So moving them is a drag-and-drop into
a new release and no change anywhere else — the assets are matched by pattern
(`fpc-laz_*_amd64.deb`, `lazarus-*win64.exe`, …), so version numbers in the names do
not have to be edited. Recommended once the build is working, not before: one thing
at a time.

## The token, because this repository is private

`SG-LPK`'s workflow cannot read a private repository with its own `GITHUB_TOKEN` —
that token is scoped to `SG-LPK` alone. One of the two:

* **keep it private** and give `SG-LPK` a token: Settings → Developer settings →
  Personal access tokens → **Fine-grained tokens** → new token, repository access
  *only* `SG-Toolchain`, permission **Contents: Read-only**. Then in **`SG-LPK`** →
  Settings → **Secrets and variables → Actions** → **Repository secrets** → New,
  named exactly `TOOLCHAIN_TOKEN`.
  The three tabs that look right and are not: **Variables** (that is `vars.`, not
  `secrets.`), **Dependabot** secrets, and **Codespaces** secrets. A secret put in
  any of those is invisible to Actions, and the symptom is exactly an empty token.
* **or make this repository public** and no secret is needed at all — there is
  nothing private in it, it is four public installers. The workflow falls back to
  `GITHUB_TOKEN`, which can read any public repository.

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
