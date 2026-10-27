Open `https://__DOMAIN____PATH__` and log in with the nick and password given during
installation. The nick is the YunoHost user you chose; the password is the one
you typed.

Administration lives at `/admin` once logged in: accounts, federation peers,
and the instance settings (history retention, connection limits, login rates).
Settings take effect immediately, without a restart.

IRC clients connect to `__DOMAIN__` on port **6697 with TLS**, using the account
name as the nick and either its password or a token from the web interface
(Settings -> tokens). If the port is not reachable from outside, that is the
router/firewall, not the package -- see `doc/PRE_INSTALL.md`.

For the command line, the admin token is what authenticates:

```bash
PARLEY_TOKEN="$(yunohost app setting parley admin_token)" \
PARLEY_ENDPOINT="https://__DOMAIN____PATH__" \
    /var/www/parley/parleyctl status
```
