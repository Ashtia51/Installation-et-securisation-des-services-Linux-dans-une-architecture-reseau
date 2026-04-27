# IV. Installation du serveur web Debian 12.5 Bookworm

## 1. Configuration du serveur
Pour notre serveur Web, nous choisissons **Debian 12** pour sa légèreté, sa simplicité et son aspect Open-Source. La communauté active assure une documentation régulière, ce qui en fait un choix idéal pour cette infrastructure. La machine est créée via **VMware Workstation Pro**.

![Configuration Hardware](../images/hardware_debian12_vmware.png)

## 2. Paramétrage réseau
L'adressage IP du serveur est configuré en statique pour garantir la stabilité des services dans la DMZ.

* **Configuration de l'interface :** Adresse IP `172.16.0.3` avec DNS et passerelle pointant sur le pfSense (`172.16.0.1`).
![Configuration Réseau](../images/network_interfaces_dmz.png)

* **Résolution de noms :** Configuration du fichier `resolv.conf`.
![Fichier resolv.conf](../images/resolv.conf_dmz.png)

* **Fichier Hosts :** Définition des noms d'hôtes locaux.
![Fichier hosts](../images/hosts_dmz.png)

## 3. Gestion des paquets et mises à jour
Avant de configurer le service Web, nous préparons les sources et mettons à jour le système.

* **Sources.list :** Mise à jour des dépôts officiels.
![Dépôts Debian](../images/sourceslist_dmz.png)

* **Mises à jour système :** Exécution de l'update et de l'upgrade. Le système est déjà à jour suite à la configuration des sources.
![Update OS](../images/update_os_dmz.png)

## 4. Mise en service d'Apache2
Le service **Apache2** a été pré-installé lors de l'assistant d'installation initial. Nous validons son bon fonctionnement avant la personnalisation.

* **Statut du service :**
![Statut Apache2](../images/systemctl_status_apache2.png)

* **Contenu par défaut :** Localisation du fichier racine dans `/var/www/html/index.html`.
![Fichier par défaut](../images/default_html.png)

## 5. Personnalisation du site et tests d'accès
Pour finaliser l'installation, nous modifions la page d'accueil pour valider le flux complet.

1. **Génération HTML :** Utilisation d'un générateur de balisage pour le design.
![Générateur HTML](../images/html_generator.png)

2. **Modification :** Injection du code dans le fichier `index.html`.
![Modif Index.html](../images/indexhtml_modif.png)

3. **Validation de connexion :**
   * Test initial depuis le réseau WAN (lors de la création du serveur).
   ![Test Connexion WAN](../images/connect_nat_web.png)
   * Test final depuis une machine cliente du LAN après configuration de la DMZ.
   ![Test Connexion DMZ](../images/connect_web_dmz.png)
