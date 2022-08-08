# Projet 10 OpenClassRooms - Aws et CloudFormation

* Exemple d'un script d'automatisation AWS pour la création d'une structure complète comprenant des EC2, un RDS, un ELB, un AutoScaling, un BucketS3 et un EFS. Ce projet a été créé dans le but de se former à la création de fichier Cloudformation d'automatisation au format YAML, ce projet fait partit de la formation d'Administrateur Système et Cloud d'OpenClassRooms.

## _Paramétrage nécessaire pour le bon fonctionnement de ce script_
Pour ce projet une VM VPN est à créée (*VM serveur*). J'ai utilisée le VPN Wireguard pour ce projet, vous aurez également besoin d'un serveur de fichier. (*__Ici, j'utilise un environnement sous Debian 11 pour mes deux machines.__*)

### __Création de notre VM VPN Wireguard__
```
apt install wireguard
```
*Génération des clés privée et publique de notre serveur.*
```
wg genkey | tee /etc/wireguard/wg-private.key | wg pubkey | tee /etc/wireguard/wg-public.key
```
*Affichez vos clés après les avoir générées afin de pouvoir les utiliser pour votre serveur VPN et votre client*
```
cat /etc/wireguard/wg-private.key
```
