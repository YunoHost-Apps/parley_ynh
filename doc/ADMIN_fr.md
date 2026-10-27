## Ligne de commande

`parleyctl` est le client d'administration. Il s'authentifie avec le jeton
d'administration de l'instance, que le paquet a généré à l'installation :

```bash
cd /var/www/parley
export PARLEY_TOKEN="$(yunohost app setting parley admin_token)"
export PARLEY_ENDPOINT="https://__DOMAIN____PATH__"

./parleyctl status
./parleyctl accounts list
./parleyctl accounts create bob -password-stdin
./parleyctl settings list
```

`parleyctl <commande> -h` documente le reste : comptes, jetons, pairs, bannissements,
rotation de clé et recherche plein texte dans l'historique de l'instance.

## Journaux

La sortie du serveur va dans `/var/log/parley/parley.log` et dans le journal
(`journalctl -u parley`). La configuration logrotate est créée par le paquet.

## Fédération

Les pairs s'ajoutent instance par instance et peuvent être bloqués entièrement,
ou par utilisateur avec des masques :

```bash
./parleyctl peers add autre.example
./parleyctl peers list
```

Un pair qui répond apparaît comme `linked` ; c'est aussi le moment où les messages
commencent à circuler — un utilisateur d'ici rejoint un canal global et apparaît
sur l'autre instance sous la forme `pseudo:domaine`.

Par défaut la politique est `open` : toute instance qui se présente peut se
fédérer, dans la limite de `federation.new_peer_rate`. Si cette instance ne doit
parler qu'aux instances que vous avez nommées :

```bash
./parleyctl settings set federation.policy allowlist
```

Mesuré sur deux instances : avec le pair dans la liste les messages continuent
d'arriver, une fois le pair retiré le message de la même instance est ignoré, et
le remettre rétablit la livraison. Le réglage est donc une barrière vivante sur
qui peut livrer, pas une poignée de main unique.

**Certificats.** La fédération sortante vérifie le certificat TLS du pair, ce qui
est souhaitable sur un vrai domaine et ne pose problème que lorsque les deux côtés
sont en auto-signé. Il existe un drapeau pour cela — `parleyd -ca <bundle>` — mais
aucune variable d'environnement : il ne peut donc pas venir de `parley.env`.
Ajoutez un drop-in systemd avec `ExecStart=` puis le nouveau
`ExecStart=<install_dir>/parleyd -ca …`, et donnez au fichier le mode 644 (le
répertoire des certificats de YunoHost est réservé à root).

L'identité de l'instance, c'est son domaine, plus une paire de clés stockée dans
`parley.db`. **`parleyctl key export -y` affiche la clé privée** ; gardez-en une
copie ailleurs si l'instance compte, car la perdre revient à apparaître comme une
nouvelle instance aux yeux des pairs.

## Changer de domaine n'est pas pris en charge

Ce paquet exclut volontairement le test `change_url` : le domaine est l'identité
de l'instance dans la fédération. Un nouveau domaine est une nouvelle instance
pour tous les pairs, même si la base et ses clés survivent.

## Supprimer l'application supprime l'instance

`yunohost app remove parley` supprime aussi `/home/yunohost.app/parley`, c'est-à-dire
l'instance entière — `parley.db` (comptes), `history.db` (messages),
`identity.key` (la clé de fédération), `vapid.key` (notifications) et `uploads/`.
Rien de tout cela ne survit à la suppression : sauvegardez d'abord.

```bash
yunohost backup create --apps parley
```

## Qui peut se connecter

L'interface web est derrière la connexion YunoHost (`sso = true`) : ssowat
authentifie la requête et nginx transmet l'identité dans `Ynh-User`,
`Ynh-User-Email` et `Ynh-User-Fullname`, que parleyd accepte parce que
`PARLEY_TRUSTED_PROXIES=127.0.0.1`.

Deux conséquences à connaître :

* **Le portail protège l'interface, pas la fédération.** `/api`, `/inbox`,
  `/.well-known` et `/avatars` sont des permissions YunoHost distinctes, ouvertes
  aux visiteurs. Si quelqu'un « range » en mettant toute l'application derrière
  `all_users`, l'instance continuera de répondre aux navigateurs et cessera de se
  fédérer : les pairs recevront une redirection vers la page de connexion au lieu
  d'une vérification de signature. parleyd authentifie malgré tout sa propre API
  avec des jetons, donc « visiteurs » signifie « pas de session YunoHost », pas
  « pas d'authentification ».
* **Parley garde sa propre session.** Visiter `/` avec une session du portail
  affiche le lien de connexion tant que `/login` n'a pas été ouvert une fois ;
  cette page voit l'en-tête de confiance et connecte la personne sans mot de
  passe. Le mot de passe local fonctionne toujours, ce qui est utile en ligne de
  commande et pour tester sans le portail.

Le groupe autorisé à utiliser l'interface est la permission `main` de
l'application :

```bash
yunohost user permission list | grep parley
yunohost user permission update parley.main --add-group visitors   # ou un groupe
```

## Connecter un client IRC

Le paquet fait de nginx un terminateur TLS devant l'écouteur IRC de Parley et
publie IRC sur le port **6697** :

```
serveur :  __DOMAIN__
port :     6697
TLS :      oui (obligatoire)
pseudo :   le nom du compte
mot de passe : celui du compte, ou un jeton (Réglages -> jetons)
```

`parleyctl me tokens create -label "irssi sur le portable"` affiche un jeton pour
le compte derrière le jeton d'administration ; l'interface web fait la même chose
avec un bouton.

Si 6697 est déjà pris sur cette machine, déplacez-le puis lancez la mise à jour :

```bash
yunohost app setting parley irc_port 6698
yunohost app upgrade parley
```

**Comment c'est câblé, pour que personne n'ait à le redécouvrir.** nginx
n'accepte qu'un seul bloc `stream {}` par configuration : `conf/parley-stream.conf`
génère donc cette enveloppe une seule fois vers
`/etc/nginx/modules-enabled/parley-stream.conf` — ce répertoire est inclus au
premier niveau de nginx, alors que `conf.d` est le contexte `http {}` où un bloc
stream est une erreur de syntaxe. Chaque instance dépose ensuite un simple
`server {}` dans `/etc/nginx/stream.d/<app>.conf`, que l'enveloppe inclut.
L'enveloppe est créée par la première instance et supprimée par la dernière.

Parley écoute lui-même sur un port loopback et n'accepte l'en-tête PROXY que des
adresses listées dans `PARLEY_IRC_PROXIES` — le paquet y met `127.0.0.1`. Les deux
moitiés sont nécessaires : avec `proxy_protocol on` mais sans cette liste, Parley
lit la ligne PROXY comme une commande IRC et répond
`451 * :You have not registered` avant la vraie conversation, ce qui ressemble à
un client cassé plutôt qu'à un proxy mal configuré.

**Une seconde instance sur la même machine** prend le port libre suivant, pour
l'écouteur public comme pour celui de loopback, puisque 6697 et 6667 sont déjà
pris : l'avertissement d'installation nomme la paire retenue et les réglages de
l'instance la conservent (`irc_port`, `irc_local_port`).
