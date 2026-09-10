# Publish to FlatPark

**Ship your GitHub release as a Flatpak — automatically.**

Add five lines to your release workflow and your Linux users get the Flatpak
update the same day you publish a release. No tokens to manage, no Flatpak
manifest to write, no build infrastructure to run. [FlatPark](https://flatpark.org)
maintains the package; this action just tells it the moment your release goes live.

```yaml
# .github/workflows/release.yml
on:
  release:
    types: [published]

jobs:
  flatpak:
    runs-on: ubuntu-latest
    # A FlatPark hiccup should never fail your release.
    continue-on-error: true
    steps:
      - uses: flatpark/publish-action@v1
        with:
          app-id: com.example.MyApp
```

That's it, and it leaves you nothing to chase. Within minutes FlatPark
re-resolves your latest release, unpacks the new artifact the way an install
does, and — when that works — merges and publishes it without waiting for a
maintainer. The new version is live on the
[FlatPark Flatpak repository](https://flatpark.org/setup), where
`flatpak update` delivers it to every installed user.

The one case that stops there is an artifact that no longer unpacks — usually
a release that was repackaged, a renamed launcher or a relocated binary. Then
the version already published stays up, untouched, and someone at FlatPark
picks it up by hand.

## Requirements

Your app must already be listed on FlatPark. Not listed yet?
[Open an issue](https://github.com/flatpark/flatpark/issues) or a PR at
[flatpark/flatpark](https://github.com/flatpark/flatpark) — packaging an app
that ships Linux release binaries usually takes a day.

## Inputs

| Input      | Required | Default                              | Description                                              |
|------------|----------|--------------------------------------|----------------------------------------------------------|
| `app-id`   | yes      | —                                    | Your Flatpak application id on FlatPark (e.g. `com.example.MyApp`) |
| `tag`      | no       | tag of the triggering release        | Release tag to publish                                   |
| `endpoint` | no       | `https://hooks.flatpark.org/release` | FlatPark release webhook                                 |

## How it works (and why it needs no secrets)

The action sends `{app_id, tag, repository}` to FlatPark's webhook. The
webhook trusts none of it at face value — it independently verifies that:

1. the app id is listed in FlatPark's public catalog,
2. the catalog registers **this repository** as the app's upstream, and
3. the tag really exists as a published release here on GitHub.

Only then does FlatPark trigger its own update pipeline, using its own
credentials. Your workflow never holds a FlatPark token, and nobody can use
this endpoint to publish releases for a repository they don't control.

Without this action your app still updates — FlatPark polls upstream releases
daily and ships them the same automatic way. This action turns that poll into
a push, so updates land in minutes instead of up to a day later.

## Why distribute on Flatpak at all?

Flatpak is the closest thing desktop Linux has to a universal package: one
build runs on Ubuntu, Fedora, Arch, openSUSE, the Steam Deck and everything
in between, with sandboxing and automatic updates built in. FlatPark
specializes in apps that ship official Linux binaries (AppImage, deb, zip,
tarball) but aren't on Flathub — it repacks your existing release artifacts,
so there is nothing new for you to build.

## License

MIT
