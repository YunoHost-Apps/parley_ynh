Open `https://__DOMAIN____PATH__` and log in with the nick and password given during
installation. The nick is the YunoHost user you chose; the password is the one
you typed.

Administration lives at `/admin` once logged in: accounts, federation peers,
and the instance settings (history retention, connection limits, login rates).
Settings take effect immediately, without a restart.

IRC clients cannot connect yet in this version -- see `doc/PRE_INSTALL.md`.
The tokens the web interface mints are for IRC clients and will start working
when the package gains a TLS terminator for the IRC listener.

For the command line, the admin token is what authenticates:

```bash
PARLEY_TOKEN="$(yunohost app setting parley admin_token)" \
PARLEY_ENDPOINT="https://__DOMAIN____PATH__" \
    /var/www/parley/parleyctl status
```
