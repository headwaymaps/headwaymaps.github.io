<https://maps.earth> is a planet scale demo site of the [Headway](https://github.com/headwaymaps/headway) mapping application.

This repository is a meta-site, hosted at about.maps.earth.

Changes merged here will be automatically published to <https://about.maps.earth> via our [ci config](./.github/workflows/publish.yml).

## Building

The site is built using [hugo](https://gohugo.io), install it if you haven't already.

```
# e.g. on macos
brew install hugo
```

Install the theme.

```
git submodule update --init
```

Start the development server (and include any draft posts).
```
hugo server --buildDrafts
```

