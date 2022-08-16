---
title: "Run it yourself"
weight: 2
header_menu: true
---

[maps.earth](https://maps.earth/) is a planet-scale installation of [Headway](https://github.com/headwaymaps/headway), but you can easily set up your own server on a smaller scale for your own personal use by running just a few commands.

```
$ earthly -P +build --area=Amsterdam --country=NL
$ cp .env-example .env
$ docker-compose up -d
```

That's all it takes to bring up a fully functional maps server for the Amsterdam metro area, including place and address search, and routing via car, bicycle and on foot. Other areas require small tweaks to the `.env` file. More details are available in the project's [BUILD.md](https://github.com/headwaymaps/headway/blob/main/BUILD.md) file.
