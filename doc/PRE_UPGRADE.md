Downgrades are not supported: `parleyd` migrates its database forward on start
and an older binary refuses to open a newer one. Keep the backup taken before
the upgrade if there is any chance you will need to go back.
