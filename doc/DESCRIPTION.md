Parley is a chat server that speaks plain IRC. Any IRC client can connect to
it, and instances federate with each other: somebody on one server can join a
channel on another, with the remote user appearing as `nick:domain`.

The web interface exists to administer the instance and to mint the tokens IRC
clients log in with. It is deliberately not a chat client -- chatting is the
job of an IRC client.

This package installs `parleyd` (the server) and `parleyctl` (the command line
client used for administration) from the upstream binary release.
