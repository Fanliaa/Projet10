Aws et CloudFormation

*Exemple d'un script d'automatisation AWS pour la création d'une structure complète comprenant des EC2, un RDS, un ELB, un AutoScaling, un BucketS3 et un EFS. Ce projet a été créé dans le but de se former à la création de fichier Cloudformation d'automatisation au format YAML, ce projet fait partit de la formation d'Administrateur Système et Cloud d'OpenClassRooms.*

## __Paramétrage nécessaire pour le bon fonctionnement de ce script__
Pour ce projet une VM VPN et Intranet sont à créer chez vous (*VM serveur*). J'ai utilisée le VPN WireGuard pour ce projet.(*__Ici, j'utilise un environnement sous Debian 11 pour mes deux machines, les machines Wordpress sur AWS sont des Amazon Linux.__*)

### __Création de notre VM VPN Wireguard__
*Ces commandes sont à faire sur vos deux VM (client et serveur).*
```
apt install wireguard
```
*Génération des clés privée et publique de notre serveur.*
```
wg genkey | tee /etc/wireguard/wg-private.key | wg pubkey | tee /etc/wireguard/wg-public.key
```
*Affichez vos clés après les avoir générées afin de pouvoir les utiliser pour votre serveur VPN et votre client.*
```
cat /etc/wireguard/wg-private.key
```
*Nous allons maintenant créer notre fichier de configuration de l'interface réseau associée à WireGuard. Ici, enp0s3 est le nom de mon interface réseau, à vous de l'adapter avec la votre.*
```
nano /etc/wireguard.wg0.conf
```
*Interface Serveur:*
```
[Interface]
Address = 10.9.8.1/24
ListenPort = 51820
PrivateKey = <Clé privée de votre serveur>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE

[Peer]
PublicKey = <Clé publique de votre client>
AllowedIPs = 10.9.8.2/24, 10.0.103.0/24
PersistentKeepalive = 25
```
*Démarrons notre interface:*
```
wg-quick up wg0
```
*Vous pouvez voir l'état de votre VPN avec la commande __wg__.*
*Faisons en sorte que cette interface démarre automatiquement au démarrage de nos VM.*
```
systemctl enable wg-quick@wg0.service
```
*Pour que notre serveur VM puisse router les différents paquets entre les réseaux, nous allons activer __Ip Forwarding__*
```
nano /etc/systemctl.cong
```
*Décommentez la ligne suivante:*
```
net.ipv4.ip_forward = 1
```
*Maintenant, pour notre client VPN, voici la partie à ajouter dans le fichier .YML à l'endroit où se trouve le client VPN:*
```
    UserData:
        Fn::Base64:
          Fn::Sub: |
            #!/bin/bash
            sudo apt -y update && upgrade
            sudo apt install wireguard
            sudo wg genkey | tee /etc/wireguard/wg-private.key | wg pubkey | tee /etc/wireguard/wg-public.key
            sudo cat /etc/wireguard/wg-private.key
            sudo cat > /etc/wireguard/wg0.conf << EOF
            [Interface]
            Address = 10.9.8.2/24
            ListenPort = 51820
            PrivateKey = <Clé privée de votre client>

            [Peer]
            PublicKey = <Clé publique de votre serveur>
            Endpoint = <Ip publique de votre box>:51820
            AllowedIPs = 10.9.8.1/32, <Réseau sur lequel se trouve vos vm chez vous>/24
            PersistentKeepalive = 25
            EOF
            sudo wg-quick up wg0
            sudo systemctl enable wg-quick@wg0.service
```
Le paramétrage de notre VPN est terminé. Pour vérifier que la connexion soit bien établie, vous pouver faire un __wg__, si l'option *latest handshake* apparraît, s'est que la connexion entre les deux machine c'est bien effectuée.

### __Création de notre serveur de fichier NFS__
```
apt install nfs-kernel-server
mkdir /votre/dossier/partagé
chmod 764 /votre/dossier/partagé
ip route add 10.9.8.0/24 via <ip locale de votre vpn serveur>
ip route add 10.0.103.0/24 via <ip locale de votre vpn serveur>
systemctl enable nfs-server.service
nano /etc/exports
```
*Dans ce fichier, vous aller ajouter cette ligne à la fin de votre fichier:*
```
/votre/dossier/partagé  <ip locale de votre vpn serveur>(rw,sync,no_wdelay)
```
*Les options entre parenthèses sont à adapter celon vos besoins.*
Le paramétrage de votre serveur de fichier est terminé. Il ne vous reste plus qu'a modifier votre fichier YML par rapport à vos besoins.
