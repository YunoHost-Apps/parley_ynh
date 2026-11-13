* A domain name, since the instance's identity in the federation is its
  domain and it cannot be changed later without becoming a different instance.
* About 150 MiB of disk: the server and its command line client are Go
  binaries of roughly 35 MiB each.
 
**IRC needs one thing from you after installation: a reachable port.** The
package terminates TLS in front of Parley's IRC listener and serves IRC on
**port 6697**, so clients connect with TLS on that port. If this instance is
behind a router or a firewall, that port has to be forwarded or opened
elsewhere — the package cannot do that part. The port is a setting
(`yunohost app setting parley irc_port`) if 6697 is taken.
