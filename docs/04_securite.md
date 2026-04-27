
# V. Sécurisation du serveur (Hardening SSH)

Après avoir isolé le serveur au niveau réseau, nous appliquons les bonnes pratiques de cybersécurité directement sur le système pour limiter la surface d'attaque.

## 1. Changement du port SSH par défaut
Le port 22 est la cible prioritaire des robots de scan et des attaques par force brute. Nous modifions le port d'écoute pour le **5091**.

![Modification du port SSH](../images/rename_ssh.png)

## 2. Authentification par clés asymétriques
Pour supprimer le risque lié aux mots de passe (souvent trop simples ou vulnérables au brute-force), nous mettons en place une authentification par paire de clés (publique/privée).

### A. Génération des clés
Utilisation de **PuTTYgen** pour générer une paire de clés robuste.
![Génération des clés](../images/asymetric_keys.png)

### B. Sauvegarde sécurisée
Il est impératif de conserver la clé privée en lieu sûr et de ne jamais la partager.
![Sauvegarde des clés](../images/keys_saved.png)

## 3. Déploiement de la clé publique
Nous utilisons **WinSCP** pour transférer la clé publique sur le serveur Debian. Celle-ci est ajoutée au fichier `authorized_keys`.

![Transfert WinSCP](../images/install_public_key.png)

* **Vérification sur le serveur :** Nous confirmons la présence de la clé dans le répertoire sécurisé de l'utilisateur (ici `/root/.ssh/`).
![Vérification RSA](../images/verif_rsa.png)

## 4. Désactivation des accès vulnérables
Une fois l'accès par clé validé, nous durcissons la configuration du service SSH (`sshd_config`) :
* **Interdiction du login Root :** Pour forcer l'usage d'un utilisateur standard ou sécurisé.
* **Désactivation des mots de passe :** Seule l'authentification par clé est désormais autorisée.

![Désactivation Login Root](../images/noroot_keyok.png)
![Désactivation Mots de passe](../images/no_passwd.png)

## 5. Test de connexion finale
Pour faciliter la connexion, nous importons la clé privée dans **Pageant** (agent SSH de PuTTY). L'authentification se fait désormais de manière transparente et sécurisée, sans qu'aucun mot de passe ne soit demandé ou ne circule sur le réseau.

* **Chargement de la clé :**
![Pageant Private Key](../images/pageant_private_key.png)

* **Résultat de la connexion :**
![Authentification réussie](../images/authentified_key.png)

## 6. Installation et configuration de FirewallD

Pour renforcer la sécurité locale du serveur (défense en profondeur), nous installons et configurons **FirewallD**. Ce pare-feu interne permet de ne laisser entrer que les flux strictement nécessaires au fonctionnement du service Web et de l'administration.

### A. Installation du service
Le paquet est installé via le gestionnaire de paquets APT.
![Installation FirewallD](../images/install_firewallD.png)

### B. Configuration des ports autorisés
Par défaut, FirewallD bloque tout le trafic entrant non autorisé. Nous ouvrons spécifiquement les ports nécessaires :

* **Port SSH personnalisé (5091) :** Autorisation de l'administration sécurisée.
![Ouverture Port 5091](../images/firewallD_ssh.png)

* **Port 80 (HTTP) :** Pour l'accès au site Web.
![Ouverture Port 80](../images/firewallD_80.png)

* **Port 443 (HTTPS) :** Pour les futures connexions sécurisées.
![Ouverture Port 443](../images/firewallD_443.png)

### C. Vérification de la configuration
Nous listons les services et ports actifs pour confirmer que la politique de "Default Deny" est bien en place pour tout le reste du trafic.
![Liste des règles FirewallD](../images/firewallD_liste.png)

> **Note :** Les règles ont été testées immédiatement. Elles ont ensuite été passées en mode permanent (via l'option `--permanent`) pour persister après un redémarrage du serveur.

## 7. Protection contre les intrusions avec Fail2Ban

Même avec un port SSH modifié et une authentification par clé, le serveur peut subir des tentatives de connexion automatisées. Nous installons **Fail2Ban** pour détecter ces comportements et bannir dynamiquement les adresses IP malveillantes.

### A. Installation du service
Nous installons le paquet pour permettre au serveur d'analyser les fichiers de logs en temps réel.
![Installation Fail2Ban](../images/fail2ban_install.png)

### B. Configuration et surveillance du service
Une fois installé, nous vérifions que le service est actif. Fail2Ban crée des "jails" (prisons) pour surveiller des services spécifiques. On constate ici qu'il cible correctement notre service SSH.
![Statut Fail2Ban SSH](../images/fail2ban_status_ssh.png)

### C. État de la "Jail" SSH
En consultant le statut détaillé de la prison SSH, nous pouvons voir les statistiques de bannissement. Cela nous permet de confirmer que le moteur de détection est opérationnel et prêt à bloquer toute tentative d'intrusion.
![Statut Jailed SSH](../images/fail2ban_jailed_status.png)

> **Note :** Fail2Ban a été configuré pour surveiller notre port personnalisé (5091). En cas de multiples échecs d'authentification, l'IP source est automatiquement ajoutée aux tables de blocage de notre pare-feu local pour une durée déterminée.

## 8. Supervision du serveur avec Netdata

Pour assurer la maintenance opérationnelle du serveur, nous avons installé **Netdata**. Cet outil de monitoring en temps réel nous permet d'analyser les performances du système (CPU, RAM, Disque) ainsi que l'activité réseau et les performances du service Apache2.

### Pourquoi Netdata ?
* **Rapidité :** Installation et configuration quasi-instantanées.
* **Interface intuitive :** Dashboard moderne accessible via navigateur web.
* **Faible consommation :** Optimisé pour ne pas impacter les ressources du serveur web.
* **Détails granulaires :** Analyse précise du hardware et des services software.

### Aperçu du Dashboard
Depuis une machine du LAN, nous accédons à l'interface de supervision pour vérifier la stabilité de la DMZ.

![Tableau de bord Netdata](../images/netdata_dashboard.png)

> **Note :** L'accès à l'interface Netdata est sécurisé et limité aux adresses IP autorisées afin de ne pas exposer les métriques du serveur sur le réseau public.
