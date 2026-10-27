Ouvrez `https://__DOMAIN____PATH__`. L'instance est derrière la connexion
YunoHost : vous vous connectez avec votre compte YunoHost, et la première visite
de `/login` rattache automatiquement le compte Parley portant le même pseudo —
aucun mot de passe Parley à saisir.

Le mot de passe fourni pendant l'installation fonctionne toujours sur `/login`
pour le compte du même nom, ce qui reste utile en ligne de commande et pour
tester l'instance sans passer par le portail.

L'administration se trouve sur `/admin` une fois connecté : comptes, pairs de
fédération et réglages de l'instance (rétention de l'historique, limites de
connexion, cadence des connexions). Les réglages prennent effet immédiatement,
sans redémarrage.

Les clients IRC se connectent à `__DOMAIN__` sur le port **6697 en TLS**, avec le
nom du compte comme pseudo et soit son mot de passe, soit un jeton créé dans
l'interface web (Réglages -> jetons). Si le port n'est pas joignable depuis
l'extérieur, cela relève du routeur ou du pare-feu, pas du paquet — voyez
`doc/PRE_INSTALL.md`.

En ligne de commande, c'est le jeton d'administration qui authentifie :

```bash
PARLEY_TOKEN="$(yunohost app setting parley admin_token)" \
PARLEY_ENDPOINT="https://__DOMAIN____PATH__" \
    /var/www/parley/parleyctl status
```
