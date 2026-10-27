* A domain name, since the instance's identity in the federation is its
  domain and it cannot be changed later without becoming a different instance.
* About 150 MiB of disk: the server and its command line client are Go
  binaries of roughly 35 MiB each.
* **A dedicated (sub)domain, with the path left as `/`.** Parley's web
  interface uses absolute paths (`/assets`, `/login`, `/theme`,
  `/site.webmanifest`) and the binary has no base-path setting, so a subpath
  install would answer on the page and then 404 on every asset. The install
  script refuses that instead of producing a half-broken instance.

**IRC needs one thing from you after installation: a reachable port.** The
package terminates TLS in front of Parley's IRC listener and serves IRC on
**port 6697**, so clients connect with TLS on that port. If this instance is
behind a router or a firewall, that port has to be forwarded or opened
elsewhere — the package cannot do that part. The port is a setting
(`yunohost app setting parley irc_port`) if 6697 is taken.

The installation creates a user, `/var/www/parley` for the binaries and
`/home/yunohost.app/parley` for the instance database, and asks for a password
for the administrator account.
