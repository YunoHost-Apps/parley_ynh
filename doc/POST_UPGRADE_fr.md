Le paquet a remplacé les binaires, régénéré `/var/www/parley/parley.env` et
l'unité systemd, puis redémarré le service. La base n'a pas été touchée :
`parleyd` applique ses propres migrations au démarrage.

**Les modifications manuelles de `parley.env` sont perdues**, puisque le fichier
est de nouveau généré à partir des modèles du paquet. Les réglages de l'instance
se font dans l'interface d'administration ou avec `parleyctl settings`, qui les
stockent dans la base et survivent aux mises à jour.
