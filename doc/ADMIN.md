## Command line

`parleyctl` is the administration client. It authenticates with the instance
admin token, which the package generated at install time:

```bash
cd /var/www/parley
export PARLEY_TOKEN="$(yunohost app setting parley admin_token)"
export PARLEY_ENDPOINT="https://__DOMAIN__"

./parleyctl status
./parleyctl accounts list
./parleyctl accounts create bob -password-stdin
./parleyctl settings list
```

`parleyctl <command> -h` documents the rest: accounts, tokens, peers, bans,
key rotation and full-text search over the instance history.

## Logs

Server output goes to `/var/log/parley/parley.log` and to the journal
(`journalctl -u parley`). The logrotate configuration is created by the
package.

## Federation

Peers are added per instance and can be blocked entirely, or per user with
masks:

```bash
./parleyctl peers add other.example
./parleyctl peers list
```

The instance's own identity is its domain, plus a key pair stored in
`parley.db`. **`parleyctl key export -y` prints the private key**; keep a copy
somewhere else if the instance matters, because losing it means peers see a
new instance.

## Changing the domain is not supported

This package excludes the `change_url` test on purpose: the domain is the
instance's identity in the federation. A new domain is a new instance as far
as every peer is concerned, even though the database and its keys survive.

## Removing the application deletes the instance

`yunohost app remove parley` also removes `/home/yunohost.app/parley`, which is
`parley.db`: accounts, federation keys and history. Back up first:

```bash
yunohost backup create --apps parley
```

## When IRC is added later

Parley speaks plain text on the IRC listener and expects TLS in front of it. If
you put a terminator (nginx `stream`, HAProxy, ...) in front, read
`docs/PROXY.md` upstream first: the proxy address must be listed as an
`irc_proxies` instance setting **and** the proxy must send the PROXY header.
One without the other breaks connections, and the address to list is the one on
the socket, not the one you assume the proxy has.
