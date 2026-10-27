* Un nom de domaine, puisque l'identité de l'instance dans la fédération est son
  domaine et qu'il ne pourra plus changer sans devenir une autre instance.
* Environ 150 Mio d'espace disque : le serveur et son client en ligne de commande
  sont deux binaires Go d'environ 35 Mio chacun.
* **Un (sous-)domaine dédié, avec le chemin laissé à `/`.** L'interface web de
  Parley utilise des chemins absolus (`/assets`, `/login`, `/theme`,
  `/site.webmanifest`) et le binaire n'a aucune option de chemin de base : une
  installation dans un sous-chemin répondrait sur la page puis renverrait 404 sur
  chaque ressource. Le script d'installation refuse ce cas plutôt que de produire
  une instance à moitié cassée.

**IRC demande une chose de votre part après l'installation : un port joignable.**
Le paquet termine le TLS devant l'écouteur IRC de Parley et sert IRC sur le
**port 6697** ; les clients s'y connectent donc en TLS. Si cette instance est
derrière un routeur ou un pare-feu, ce port doit être redirigé ou ouvert
ailleurs — le paquet ne peut pas s'en occuper. Le port est un réglage
(`yunohost app setting parley irc_port`) si 6697 est déjà pris.

**L'interface web est derrière la connexion YunoHost.** Toute personne qui doit
l'utiliser a besoin d'un compte YunoHost et doit appartenir à la permission
`main` de cette application (tous les utilisateurs par défaut). Les chemins de
fédération — `/api`, `/inbox`, `/.well-known` et `/avatars` — sont des
permissions distinctes ouvertes aux visiteurs à dessein : les pairs ne peuvent
pas détenir de session YunoHost, et fermer ces chemins rendrait l'instance
injoignable depuis le réseau tout en ayant l'air en bonne santé.

L'installation crée un utilisateur, place les binaires dans `/var/www/parley` et
la base de l'instance dans `/home/yunohost.app/parley`, et demande un mot de
passe pour le compte administrateur.
