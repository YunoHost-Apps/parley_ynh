## Command line

`parleyctl` is the administration client. It authenticates with the instance
admin token, which the package generated at install time:

```bash
cd /var/www/parley
export PARLEY_TOKEN="$(yunohost app setting parley admin_token)"
export PARLEY_ENDPOINT="https://__DOMAIN____PATH__"

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

`yunohost app remove parley` also removes `/home/yunohost.app/parley`: that is
the whole instance — `parley.db` (accounts), `history.db` (messages),
`identity.key` (the federation key), `vapid.key` (push) and `uploads/`. Nothing
of it survives the removal, so back up first:

```bash
yunohost backup create --apps parley
```

## Who may log in

The web interface is behind the YunoHost login (`sso = true`): ssowat
authenticates the request and nginx forwards the identity in `Ynh-User`,
`Ynh-User-Email` and `Ynh-User-Fullname`, which parleyd trusts because
`PARLEY_TRUSTED_PROXIES=127.0.0.1`.

Two consequences worth knowing:

* **The portal gate covers the interface, not the federation.** `/api`, `/inbox`,
  `/.well-known` and `/avatars` are separate YunoHost permissions open to
  visitors. If someone "tidies up" by putting the whole app behind `all_users`,
  the instance keeps answering browsers and stops federating — peers get a login
  redirect instead of a signature check. parleyd still authenticates its own API
  with bearer tokens there, so "visitors" means "no YunoHost session", not "no
  authentication".
* **Parley keeps its own session.** Visiting `/` with a portal session shows the
  Log in link until `/login` is hit once; that page sees the trusted header and
  logs the person in without a password. The local password still works, which
  is what you want on the command line and when testing without the portal.

The group that may use the interface is the app's `main` permission:

```bash
yunohost user permission list | grep parley
yunohost user permission update parley.main --add-group visitors   # or a group
```

## Connecting an IRC client

The package runs nginx as a TLS terminator in front of Parley's IRC listener and
publishes IRC on port **6697**:

```
server:   __DOMAIN__
port:     6697
TLS:      yes (required)
nick:     the account name
password: the account password, or a token from Settings -> tokens
```

`parleyctl me tokens create -label "irssi on laptop"` prints a token for the
account behind the admin token; the web interface does the same thing with a
button.

If 6697 is already taken on this machine, move it and run the upgrade:

```bash
yunohost app setting parley irc_port 6698
yunohost app upgrade parley
```

**How it is wired, so nobody has to rediscover it.** `conf/parley-irc.conf`
renders to `/etc/nginx/modules-enabled/parley-irc.conf`, because that directory
is included at nginx's top level and a `stream {}` block cannot live in
`conf.d` (that is the `http {}` context). Parley itself listens on
`127.0.0.1:6667` and only accepts the PROXY header from the addresses listed in
`PARLEY_IRC_PROXIES` — the package sets `127.0.0.1`. Both halves are required:
with `proxy_protocol on` but without that list, Parley parses the PROXY line as
an IRC command and answers `451 * :You have not registered` before the real
conversation, which looks like a broken client rather than a broken proxy.
