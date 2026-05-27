# Chapitre 4 - Pare-feux

## 1. Ce que le cours veut enseigner

Le support `2526_Chapitre 4_VF.pptx` traite 5 blocs :

1. presentation generale et fonctionnement du firewall
2. politiques de securite et regles de filtrage
3. classification des firewalls
4. topologies reseaux : bastion host, DMZ, multi-homed firewall
5. pratique : ecriture de regles et ACL

Le probleme du support n'est pas qu'il soit faux. Le probleme est qu'il passe vite d'une definition generale a des categories techniques, alors que l'examen demande surtout :

- lire une regle
- comprendre LAN / WAN / DMZ
- deduire ce qui est autorise ou bloque
- proposer un equipement complementaire comme IPS
- reconnaitre un VPN et son type

Donc pour l'examen, il faut reconstruire le chapitre autour des usages, pas autour des classifications.

---

## 2. Ce qui tombe vraiment dans les examens

En lisant les sujets et corrections du dossier `examen`, les themes qui reviennent sont :

### A. Lire une table de regles firewall

Questions frequentes :

- donner la politique textuelle correspondant a des regles
- dire quelle regle traite un paquet
- dire si le paquet est accepte ou refuse
- completer une table de regles

Ce point est le plus important.

### B. Comprendre l'architecture LAN / WAN / DMZ

Questions frequentes :

- role du LAN
- role de la DMZ
- pourquoi un serveur public va en DMZ
- qui peut communiquer avec qui

### C. Regles manquantes et logique de filtrage

Questions frequentes :

- existe-t-il des regles manquantes ?
- faut-il une regle de refus par defaut ?
- comment traduire une politique textuelle en ACL ?

### D. Limites du firewall et ajout d'un autre equipement

Questions frequentes :

- certaines attaques ne sont pas arretees par le firewall
- quel equipement ajouter ?
- ou le placer ?
- quel type d'architecture cela represente ?

Reponse recurrente : `IPS` ou `NIDS`, selon l'enonce.

### E. VPN

Questions frequentes :

- proposer une connexion securisee pour les employes a distance
- dire si c'est `client-to-site` ou `site-to-site`
- dire ce qui assure la securite
- distinguer tunnel / transport dans IPSec

---

## 3. Definitions minimales a connaitre

### Firewall

Un firewall est un equipement materiel ou logiciel qui filtre le trafic entre des zones de confiance differentes.

Il prend une decision selon des criteres comme :

- IP source
- IP destination
- port source
- port destination
- protocole
- action : `allow`, `deny`, `drop`

### LAN

`LAN` = reseau interne de l'entreprise.

C'est la zone la plus protegee.  
Elle contient en general :

- les postes des employes
- les ressources internes
- les donnees internes

### WAN

`WAN` = reseau externe, souvent Internet.

C'est la zone la moins fiable.

### DMZ

`DMZ` = zone intermediaire entre Internet et le LAN.

Elle sert a placer les serveurs qui doivent etre accessibles depuis l'exterieur sans exposer directement le reseau interne.

Exemples de serveurs qu'on met en DMZ :

- serveur Web
- serveur DNS public
- serveur de messagerie

### Idee simple de la DMZ

Sans DMZ :

`Internet -> serveur dans le LAN`

Probleme : on expose le reseau interne.

Avec DMZ :

`Internet -> serveur en DMZ`

Le LAN reste derriere une protection supplementaire.

---

## 4. Le point le plus important : comment lire une regle firewall

Une regle se lit toujours comme ceci :

`Action | Source | Port source | Destination | Port destination | Protocole`

Exemple :

`Accept | DMZ network | * | LAN network | 22 | TCP`

Lecture correcte :

- `Action = Accept` : le trafic est autorise
- `Source = DMZ network` : le trafic vient de la DMZ
- `Destination = LAN network` : il va vers le LAN
- `Port destination = 22`
- `Protocole = TCP`

Comme le port `22` correspond a `SSH`, cela veut dire :

`Les machines de la DMZ peuvent se connecter aux machines du LAN en SSH`

Si la regle suivante est :

`Drop | * | * | * | * | *`

alors tout le reste est refuse.

### Conclusion sur cet exemple

Les machines de la DMZ peuvent acceder au LAN uniquement en `SSH (TCP/22)`.  
Tous les autres acces sont bloques.

---

## 5. Difference entre allow, deny et drop

Le support mentionne les trois.

### Allow

Le paquet passe.

### Deny

Le trafic est bloque.

### Drop

Le trafic est bloque, souvent sans informer clairement l'emetteur.

Pour l'examen, beaucoup d'enseignants utilisent `deny` et `drop` presque comme des reponses de blocage.  
Ce qu'il faut surtout retenir :

- `allow` = autoriser
- `deny/drop` = bloquer

---

## 6. Les deux politiques de securite

### Politique 1 : interdire tout par defaut

Formule :

`Tout ce qui n'est pas explicitement autorise est interdit`

C'est la politique la plus securisee et la plus recommandee.

En pratique :

- on autorise uniquement les flux necessaires
- tout le reste est refuse

C'est la logique de beaucoup de questions d'examen.

### Politique 2 : autoriser tout par defaut

Formule :

`Tout ce qui n'est pas explicitement interdit est autorise`

Elle est plus confortable pour les utilisateurs, mais moins securisee.

### Ce qu'il faut retenir pour l'examen

Si un tableau finit par une regle generale de type :

`Drop * * * * *`

alors on est dans une logique de `refus par defaut`.

---

## 7. Methode d'examen pour un paquet

Quand on te donne un paquet et une table de regles :

1. relever IP source
2. relever IP destination
3. relever port destination
4. relever protocole
5. lire les regles dans l'ordre
6. prendre la premiere regle qui correspond
7. si aucune ne correspond, appliquer le refus par defaut

### Erreur frequente

Ne pas repondre en regardant seulement le port.

Il faut verifier en meme temps :

- source
- destination
- protocole
- port
- ordre des regles

---

## 8. DMZ : ce qu'il faut retenir sans confusion

La DMZ n'est ni le LAN, ni Internet.

C'est une zone tampon.

### Regle memoire

- `WAN/Internet` : public, peu fiable
- `DMZ` : serveurs exposes
- `LAN` : reseau interne protege

### Regle logique

- Internet peut acceder a certains services en DMZ
- le LAN peut acceder a la DMZ selon les besoins
- la DMZ ne doit pas acceder librement au LAN
- le LAN ne doit pas etre expose directement a Internet

### Reponse-type attendue

Pourquoi mettre un serveur public en DMZ ?

Reponse :

`Pour permettre un acces depuis Internet sans exposer directement le LAN interne`

---

## 9. Ce qu'un firewall peut faire et ne peut pas faire

Le PPT donne un recapitulatif utile.

### Peut faire

- filtrer des paquets
- controler des ports et protocoles
- journaliser
- segmenter des zones
- participer a la protection des applications
- parfois assurer du NAT/NAPT et du VPN

### Ne peut pas faire seul

- stopper toutes les attaques
- remplacer un antivirus
- empecher l'ingenierie sociale
- empecher tous les abus internes

### Pourquoi c'est important a l'examen

C'est la base des questions :

- "certaines attaques ne peuvent pas etre arretees par le firewall"
- "quel equipement ajouter ?"

La reponse attendue est souvent :

- `IPS`
- ou architecture `NIDS/IDS/IPS` selon l'enonce

---

## 10. IPS, IDS, NIDS : ce qu'il faut savoir juste pour l'examen

### IDS

Detecte et alerte.

### IPS

Detecte et peut bloquer.

### NIDS

IDS place au niveau reseau.

### Idee a retenir

Le firewall filtre selon des regles connues.  
L'IDS/IPS aide quand il faut detecter ou bloquer des attaques plus evoluees.

---

## 11. VPN : ce qu'il faut retenir

Le chapitre 4 introduit le VPN comme fonctionnalite complementaire.

### Role

Assurer la confidentialite des communications a distance.

### Deux cas frequents

#### Client-to-site

Un utilisateur distant se connecte au reseau de l'entreprise.

Exemple :

- employe en teletravail

#### Site-to-site

Deux reseaux entiers sont relies entre eux.

Exemple :

- siege vers agence

### IPSec

Le cours cite `IPSec / SSL`.

Pour l'examen :

- `site-to-site` -> souvent `tunnel`
- `client-to-site` -> souvent `tunnel`, parfois `transport ou tunnel` selon l'enonce/correction

### Ce qui assure la securite

Le chiffrement.

---

## 12. NAT / NAPT

Le chapitre rappelle qu'un firewall moderne peut aussi faire :

- `NAT`
- `NAPT` ou `PAT`

Idee simple :

- le LAN utilise souvent des IP privees
- le firewall fait la traduction vers une IP publique

Ce point est moins central que les regles et la DMZ, mais il peut apparaitre dans une question de comprehension.

---

## 13. Ce qui est clair dans le PPT et ce qui l'est moins

### Ce qui est clair

- definition generale du firewall
- difference entre politique restrictive et permissive
- grandes familles de firewalls
- idee de la DMZ
- mention des fonctions complementaires : NAT, VPN, segmentation

### Ce qui est moins clair pour l'etudiant

- la transition entre theorie et questions d'examen
- la lecture pas a pas d'une regle
- la difference pratique entre LAN, WAN et DMZ
- la facon de deduire la reponse a partir d'un tableau
- les questions de type "quelle regle traite ce paquet ?"

### Conclusion pedagogique

Le support est correct sur le fond, mais il n'est pas assez guide pour les besoins de l'examen.  
Il faut ajouter une methode de lecture et plusieurs exemples corriges.

---

## 14. Resume ultra-court a memoriser

### A connaitre absolument

- `LAN` = reseau interne protege
- `WAN` = Internet / exterieur
- `DMZ` = zone intermediaire pour serveurs publics
- un firewall filtre selon `source`, `destination`, `port`, `protocole`, `action`
- politique recommandee : `interdire tout par defaut`
- `Drop * * * * *` = tout le reste est bloque
- DMZ sert a exposer un service sans exposer le LAN
- firewall ne suffit pas contre toutes les attaques -> penser `IDS/IPS`
- acces distant securise -> `VPN`

### Methode pour l'examen

Pour chaque question de tableau :

1. lire la source
2. lire la destination
3. lire le port et le protocole
4. chercher la premiere regle correspondante
5. conclure : accepte ou refuse

---

## 15. Reponse-type a ta question sur la DMZ

Si on te demande :

`C'est quoi la DMZ ?`

Tu peux repondre :

`La DMZ est une zone intermediaire entre Internet et le LAN. Elle contient les serveurs qui doivent etre accessibles depuis l'exterieur, comme un serveur Web ou DNS, afin de proteger le reseau interne.`

---

## 16. Reponse-type a une question de tableau

Question :

`Accept | DMZ network | * | LAN network | 22 | TCP`
`Drop   | *           | * | *           | *  | *`

Reponse :

`Seules les machines de la DMZ peuvent acceder au LAN en SSH sur le port 22/TCP. Tout le reste est bloque par la regle de drop.`

