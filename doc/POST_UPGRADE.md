The package replaced the binaries, regenerated `/var/www/parley/parley.env` and
the systemd unit, and restarted the service. The database was not touched:
`parleyd` applies its own migrations on start.

**Manual edits to `parley.env` are lost**, because the file is rendered from the
package templates again. Instance settings belong in the admin interface or in
`parleyctl settings`, which are stored in the database and survive upgrades.
