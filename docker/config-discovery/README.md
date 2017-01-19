# warchantua/configdiscovery

This image represents config managing server for iroha.

**This is temporal solution. It is *insecure*, because private key is also transmitted to configdiscovery service.**

To make multiple iroha instances work in single node, each of them should have `config/sumeragi.json` configuration file, which looks like this:

```json/sumeragi.json
{
  "me":{
    "ip":"172.17.0.2",
    "name":"iroha1",
    "publicKey":"jDQTiJ1dnTSdGH+yuOaPPZIepUj1Xt3hYOvLQTME3V0=",
    "privateKey":"iJy2wzgM0Ffmur2xDNlnhYK7CiAYZoup/045JXJTbUzuE9c6HeUIf7hoqtppEsZQncC1EEw+gGhboLcbMNKadw=="
  },
  "group":[
    {
      "ip":"172.17.0.2",
      "name":"iroha1",
      "publicKey":"jDQTiJ1dnTSdGH+yuOaPPZIepUj1Xt3hYOvLQTME3V0="
    },
    {
      "ip":"172.17.0.3",
      "name":"iroha2",
      "publicKey":"Q5PaQEBPQLALfzYmZyz9P4LmCNfgM5MdN1fOuesw3HY="
    },
    {
      "ip":"172.17.0.4",
      "name":"iroha3",
      "publicKey":"f5MWZUZK9Ga8XywDia68pH1HLY/Ts0TWBHsxiFDR0ig="
    },
    {
      "ip":"172.17.0.5",
      "name":"iroha4",
      "publicKey":"Sht5opDIxbyK+oNuEnXUs5rLbrvVgb2GjSPfqIYGFdU="
    }
  ]
}
```

To make automatic builds and tests it is necessary to make automatic config delivery. Configs should be automatically generated by a service, which is aware about all nodes.

This service is `configdiscovery` docker image.

There is a binary `bin/make_sumeragi`, which generates `me` section of `sumeragi.json`. It looks like this:
```json
{
    "ip":"172.17.0.2",
    "name":"iroha1",
    "publicKey":"jDQTiJ1dnTSdGH+yuOaPPZIepUj1Xt3hYOvLQTME3V0=",
    "privateKey":"iJy2wzgM0Ffmur2xDNlnhYK7CiAYZoup/045JXJTbUzuE9c6HeUIf7hoqtppEsZQncC1EEw+gGhboLcbMNKadw=="
  }
```

Right after start of iroha docker image, it executes `bin/make_sumeragi`, sends output to `configdiscovery:8000` and waits for answer (until all nodes send their `me` sections). Answer is generated `sumeragi.json` file. (You can look at for details [configure script](../build/scripts/configure.sh)).

# How to build

As usual:

```
docker build -t warchantua/configdiscovery .
```

# How to use

```
bin/make_sumeragi -o /tmp/me.json -i eth0 -n myname
cat /tmp/me.json | nc configdiscovery 8000 > config/sumeragi.json
```