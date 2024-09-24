---
layout: post
title: "The power of github actions"
categories: misc
---

I've recently started using Github actions a lot more due to my new ongoing project called [stagnant](https://github.com/legoraft/stagnant). This static site generator allows you to create a statically generated blog site (mainly to use with github pages). I wanted to create a project in rust for a long time to learn a bit more about the language. The best way to do that is by building something you'd actually use, so you'll work on it.

Now on to Github actions. When I started on this project, I just wanted to be able to upload the built site (which resides in a `site` directory) to a github pages site and be able to browse the site from github pages. A lot of these static site generators use github actions for this and I chose to take a look at [mdbook](https://github.com/rust-lang/mdBook), which had quite a clear workflow to do this. After some fiddling around with this (it took 14 commits to get it working) I realised that I was using a MacOS build on a workflow that ran on a Linux container.

Luckily, I have a linux desktop, so I just cloned the repo, built the binary and pushed it to the release. This is something I can't always do, because I do most of my coding on the run. This means that I had to find another way to get releases for multiple platforms. This is were I fell down into the Github actions rabbit hole.

I wanted to have a workflow which I can run to create release binaries for multiple platforms. Luckily, this is easily possible by creating runners for multiple platforms. Rust also has this great actions plugin that's called `houseabsolute/actions-rust-cross`. By using this I can compile rust for both of my platforms. I can also add flags and give the packages names. 

After finding out some things thanks to the [repo](https://github.com/houseabsolute/actions-rust-cross) I landed on this:

```yml
jobs:
  build:
    name: ${{ matrix.platform.os_name }}
    runs-on: ${{ matrix.platform.os }}
    strategy:
      fail-fast: false
      matrix:
        platform:
          - os_name: Linux-x86_64
            os: ubuntu-latest
            target: x86_64-unknown-linux-gnu
            bin: stagnant-x86_64-linux
          - os_name: Macos-arm
            os: macOS-latest
            target: aarch64-apple-darwin
            bin: stagnant-arm-macos

    steps:
    - uses: actions/checkout@v4
    - name: Build
      uses: houseabsolute/actions-rust-cross@v0
      with:
        command: "build"
        target: ${{ matrix.platform.target }}
        args: "--release"
        strip: true
```

This creates two runners, for macos and linux (mostly because I don't use windows). On each runner, we build the release (with the `--release` flag) and will strip any binaries if possible.

One thing that I wanted to do differently from my testing action (which is a simple action you can add with a single click) is that I chose when to run it, in stead of on every push. Luckily, this could be done with the `on: workflow_dispatch` line, which just gives you a button to run the action whenever you'd like to.

After building the packages, I just needed to upload them as an artifact and give them the correct names. This can also easily be done with the default github actions. After finding out the library and using it, this was the result:

```yml
- name: Rename binaries
      run: mv target/${{ matrix.platform.target }}/release/stagnant target/${{ matrix.platform.target }}/release/${{ matrix.platform.bin }}
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: ${{ matrix.platform.bin }}
        path: |
          target/${{ matrix.platform.target }}/release/${{ matrix.platform.bin }}
```

This just renames the binaries with the `mv` command and uploads the artifact for every platform. After this, you can just go to the workflow [run](https://github.com/legoraft/stagnant/actions/workflows/releases.yml) and download the necessary artifacts. With this, I don't have to have specific hardware just to build releases and releases can be built very quickly (a run takes <1 min). If you want to see the full file, you can check out the final [workflow](https://github.com/legoraft/stagnant/blob/main/.github/workflows/releases.yml).

When I built this. I also was building a nightly release workflow, which basically runs on every push or pull request and creates a nightly release for that. This works somewhat in the same way, but just builds the binary for linux to reduce the amount of runners. To publish a nightly release, I just update a release I've created earlier called nightly. This can easily be done with the `softprops/action-gh-release@v2` action. With this, I just needed to take the built binary and upload it to the specified tag. This is easily done:

```yml
- name: Update release
      uses: softprops/action-gh-release@v2
      with:
        tag_name: nightly
        files: ./target/release/stagnant
```

With this, I've upgraded my CI/CD pipeline for my new project and I've learned a lot of the strengths of github actions. Let's get back to the start, where I wanted to have an action to deploy my site to github pages. Luckily, mdbook had a great workflow which I could adjust a bit. You are seeing the result of this work on my site! If you'd like to take a peek under the hood, take a look at the [workflow](https://github.com/legoraft/legoraft.github.io/actions/runs/9548873972/workflow).