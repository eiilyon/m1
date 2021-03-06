*11/06/18 - Cours 6*

# Rappels sur OSI

On va travailer essentiellement sur le modèle OSI :   
**Couches logicielles** :  
7 - Transport  
6 - Présentation  
5 - Session  

**Couche charnière** :  
4 - Transport

**Couches matérielles** :  
3 - Réseau  
2 - Liaison  
1 - Physique  

1. Physique, signal, des 0 et des 1
2. Trames ethernet suivant la norme IEEE802.3  :

| P+D        | @mac dest | @mac src | Type                                                            | Data                                                                                | CRC |
| ---------- | --------- | -------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------- | --- |
| 7+1 octets | 6 octets  | 6 octets | 2 octets                                                        | Data                                                                                | CRC |
|            |           |          | Code indiquant protocole qui va prendre la suite de l'opération | Datagramme IP : il devrait nous fournir le schéma (au pire ça se trouve sur le web) | CRC |

Le datagramme IP (codé sur 32 bits) comporte un important entête composé de plusieurs parties. Dans la DATA du datagramme IP on retrouve le segment TCP (codé sur 32 bits). Il a aussi un entête puis la DATA.  

3. C'est le datagramme IP  
4. TCP ou UDP, regarde port src et port dst

### Exercice

Consignes : voir ressources cours 1

1- Adresses mac :   
src : 00 80 C8 7A 0A D8  
dst : 00 D0 59 82 2B 86  
2- Entête datagramme :  
45 00 00 40  
8B 12 40 00  
40 06 57 17  
AC 10 00 64  = IP source : 172.16.0.100  
AC 10 00 0A  = IP dest : 172.16.0.10  
3 - Version : ipdv4 (premier octet du datagramme)  
4- Longueur d'entête : 5 (IHL : deuxième octet datagramme)
5- 5 * 4 octets : 20 octets 
6- 06 : TCP  
7- 0800  
8- Port source : 110E : 4341
Port destination : 0015 : 21  
9- 6+6+2 = 14 (on compte en général pas P+D)

## L'entête TCP

Établissement de la connexion : mode consistant à n'autoriser à entrer sur un réseau uniquement le traffic internet répondant aux requêtes émises depuis le réseau local. Le routeur firewall se charge de bloquer/autoriser le traffic. 

C'est l'échange connu avec les flags en TCP pour établir une connexion :  

- SYN (avec un n° de séquence : codé sur 32 bits)  
- ACK-SYN (répond + démande synchro), (numéro d'acquittement = numéro de séquence+1, nouveau numéro de séquence)  
- ACK (numéro de séquence +1, numéro d'acquittement = nouveau numéro de séquence +1 )

Il faut donc que le paquet ait un numéro de séquence+1 pour entrer dans le réseau

### Exercice

Trames capturées par moniteur réseau. Il a ajouté des éléments qui ne nous intéressent pas (propres au moniteur)

L22 : adresse mac source  
L26 : Type   
... On entre dans datagramme IP    
L42 : Protocole  
L46 et 47 : IP sources et destination   
L54 : 0 pour les 4 premiers bits puis 0010 indiquent la levée de drapeau (SYN)  

Autre trame :  
L 90 : addresses  
..  
L124 125 : Les numéros d'acquittance et de séquence  

## La RFC1918

Request For Command  
Elle détermine pour l'IPv4 les adresses IP publiques et celles privées. Il ya un système de classes. Les privées :

| Classe | ip/masque      | plage                         |
| ------ | -------------- | ----------------------------- |
| A      | 10.0.0.0/8     | 10.0.0.1 à 10.255.255.255     |
| B      | 172.16.0.0/12  | 172.16.0.1 à 172.31.255.255   |
| C      | 192.168.0.0/16 | 192.168.0.1 à 192.168.255.255 |

Si y a des adresses privées, c'est donc qu'on a de la NAT pour pouvoir permettre de communiquer avec Internet.

Attention, SNAT en Cisco = Static NAT, ic c'est Source NAT.  
DNAT n'est pas Dymanic NAT comme en Cisco mais Destination NAT.

Construction d'une table NAT/PAT:

- IPlocale:port de base
- IP:port de destination
- IPpublique:port personnalisée
- IP:port destination  

### Exercice (4)

1.1 Parce que c'est un réseau privé

2.1 Parce que le 80 est utilisé pis faut différentier autre serveur http  
2.2 Au lieu de laisser par défaut 80 ils vont sélectioner le 4500

3.1 

---

*13/06/18 - Cours 7*

## IPTables

### NetFilter

Pre-routing : Agit sur paquet avant routage, pour permettre à un client extérieur d'accéder à un membre du réseau (change adresse IP de destination) .
Post-routing : Agit sur paquet après routage, pour permettre à un membre du réseau local d'accéder à Internet (change adresse IP source).

### TP

**iptables -t nat -A POSTROUTING -O eth1 -j MASQUERADE**  
-t table  
-o : sortie  
-j cible  
Masquerade masque les adresses privées du local. Un peu plus gourmande que commande suivante car elle va chercher l'IP sur internet  

**iptable -t nat -A POSTROUTING -O eth1 -j SNAT --tosource XX.XX.XX.XX**  
SNAT précise l'adresse IP source qu'on a côté Internet

 La config réseau du TP est pas claire - abandon du TP après deux heures à essayer de comprendre comment configurer le réseau des VM.

Il est conseillé de se faire un bloc-note avec les commandes. Pour l'exam ?

Statégie par défaut : ACCEPT

### Options

- -t pour préciser une table
- -N Créer une chaîne
- -F vide une chaine ou toute la table si pas de chaine spécifiée. On peut préciser INPUT ou OUTPUT pour filtrer les chaines à vider. La chaine continuera d'exister mais sera vide
- -X Supprime une chaine. Elle doit être vide (donc -F avant pour virer les règles)
- -L liste les règles d'une chaine ou toutes si aucune chaine spécifiée
- -A ajoute une règle; -D la supprime. On précise la chaine (PREROUTING, OUTPUT, INPUT, POSTROUTING...) au début de l'ajout/suppression de la règle. si on précise un numéro avec D on indique la ligne

**Paramètres pour ajout de règles**

- -p protocole (tcp, udp, icmp, esp...)

- -j jump : l'action a effectuer si le paquet correpond à la règle (par exemple une autre chaine)

  - SNAT : Nat source
  - DNAT : Nat destination
  - LOG : journalise tous les paquets (dans  `/var/log/messages` par défaut)
  - MASQUERADE : Camouflage (cas particulier de nat source)

- -o interface de sortie

- -m spécifie une correspondance à détecter sur le paquet (lié à une extension conntrack ?). peut utiliser les inversions si suivi de `!`

  - `state` pour checker un état ( `--state NEW`, ou ESTABLISHED, RELATED, INVALID)
  - `limit `pour spécifier des limites de temps et de connexions : `--limit 2/minute`, `--limit-burst 5`)
  - `multiport` pour spécifier plusiuers ports (jusqu'à 15) : `--source-port 22,53,80`
  - `mac` pour filtrer à partir adresse MAC (niveau 2). Cette correspondance ne s'applique qu'aux interfaces ethernet et n'est valide que dans les chaines PREROUTING, FORWARD et INPUT : `--mac-source 00:00:00:01`

- `--icmp-type` précise si le paquet est entrant ou sortant dans le cadre du protocole icmp

- `--log-level info` avec log indique le niveau de journalisation (de 0 à 8)

- `--log-prefix` puis un texte pour préfixer nos journaux.

  