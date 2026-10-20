* A domain name, since the instance's identity in the federation is its
  domain and it cannot be changed later without becoming a different instance.
* About 150 MiB of disk: the server and its command line client are Go
  binaries of roughly 35 MiB each.

**IRC is not reachable from the network in this version of the package.** The
IRC listener runs on loopback only, because Parley expects TLS to be
terminated in front of it and this package does not ship a terminator yet.
Administration over the web interface and federation over HTTPS work normally;
IRC clients cannot connect until a later version of the package adds that.

The installation creates a user, `/var/www/parley` for the binaries and
`/home/yunohost.app/parley` for the instance database, and asks for a password
for the administrator account.
