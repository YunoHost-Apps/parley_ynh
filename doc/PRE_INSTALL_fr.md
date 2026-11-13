* Un nom de domaine, puisque l'identité de l'instance dans la fédération est son
  domaine et qu'il ne pourra plus changer sans devenir une autre instance.
* Environ 150 Mio d'espace disque : le serveur et son client en ligne de commande
  sont deux binaires Go d'environ 35 Mio chacun.

**IRC demande une chose de votre part après l'installation : un port joignable.**
Le paquet termine le TLS devant l'écouteur IRC de Parley et sert IRC sur le
**port 6697** ; les clients s'y connectent donc en TLS. Si cette instance est
derrière un routeur ou un pare-feu, ce port doit être redirigé ou ouvert
ailleurs — le paquet ne peut pas s'en occuper. Le port est un réglage
(`yunohost app setting parley irc_port`) si 6697 est déjà pris.

