# Compromise Checker threat packs

Threat packs for [Compromise Checker](https://github.com/john-c-houser-tech/compromise-checker):
signed, dated files that teach the tool about one threat. A pack says which
package versions are vulnerable on each Linux release (from the vendors'
own advisories), which settings mitigate it, and which signs of attack to
look for. A pack is data, never code: nothing in it is run.

Released by NouriLoop LLC under [CC BY 4.0](LICENSE).

## Getting them

```
checker packs pull
```

downloads the index and any newer packs from this repository over HTTPS,
and checks every signature before anything is kept. `checker analyze --all`
then applies them to the evidence already collected.

Air-gapped sites copy the files across and add them by hand:

```
checker packs keys add keys.yaml        # a newer key list, if there is one
checker packs add packs/<id>/PACK.yaml  # reads PACK.yaml.minisig beside it
```

## What is here

| File | What it is | Signed by |
|---|---|---|
| `keys.yaml` | the pack keys trusted now, and any revoked | the root key |
| `index.yaml` | each pack's current version and SHA-256 hash, and any withdrawn | a pack key |
| `packs/<id>/PACK.yaml` | one pack, its current version | a pack key |

Each file's signature is beside it, as `<file>.minisig`.

## Checking a signature yourself

The tool checks every signature itself. To check one by hand, use
[minisign](https://jedisct1.github.io/minisign/) with the public key that
signed it:

```
minisign -V -P <root public key> -m keys.yaml
minisign -V -P <pack public key> -m packs/<id>/PACK.yaml
```

- **The root public key** (key id `93AB05572B87296E`) ships inside the tool.
  It is also published at
  [nouriloop.com/keys/compromise-checker-root.pub](https://nouriloop.com/keys/compromise-checker-root.pub),
  so it can be checked from two independent places.
- **The pack public keys** are listed in `keys.yaml`.

## How releases work

- **Signing:** releases are signed by hand, offline. No key is held by CI
  or by any server.
- **The index:** `index.yaml` carries a serial number that only goes up,
  so an older index can't be swapped in. It is signed again at least once
  a month, even when nothing has changed. If `checker packs pull` reports
  an index more than 30 days old, the copy you are reading from may have
  stopped updating.
- **Replacing a pack:** a pack is replaced by a newer version, never
  changed in place.
- **Withdrawing a pack:** a pack that turns out to be wrong is listed
  under `withdrawn` in the index, and the next pull retires it.
- **Replacing a key:** a pack key that leaks or is lost is revoked by a
  new `keys.yaml`, without a new release of the tool.

## Contributing

Pull requests are welcome. NouriLoop LLC reviews each one, checks every
version and date against the vendor's own page, and signs the release.
Please give each fixed version's source (the vendor's advisory or
changelog) in the pack. A pack is checked exactly as the tool reads it:

```
checker packs check PACK.yaml
```
