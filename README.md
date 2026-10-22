# PROJET_ROYAUME_MEDIEVAL
Gestion d’un royaume (DA médiévale/fantasy)


# **Projet – Gestion de royaumes**

## **1. Présentation du projet**

L'application permet aux utilisateurs de gérer leur propre royaume et de le faire évoluer au fil du temps dans une **partie multijoueur en temps réel** (jusqu'à 10 joueurs max par partie).

Chaque joueur possède un et un seul royaume caractérisé par une population, des bâtiments, des ressources et un niveau dicté par son Château.

L'objectif est de développer son royaume en utilisant ses ressources et sa main-d'œuvre de manière stratégique, d'ériger des bâtiments, de lever des armées et d'interagir avec les autres royaumes (commerce, espionnage, pillage) pour décrocher la victoire[cite: 3].

---

## **2. Session de jeu, Carte & Administration**

### **Session de jeu (`Partie`)**
* **Format :** Multijoueur en temps réel[cite: 3].
* **Capacité :** 10 joueurs maximum par session.
* **Paramètres configurables :** Vitesse de déroulement de la partie (tick rate), mode de jeu, conditions de victoire activées.

### **Carte du monde**
* **Modélisation :** Carte vectorielle 2D continue basée sur des coordonnées fixes `(positionX, positionY)`.
* **Pas de tuiles :** Aucune gestion de biomes ou d'extension de territoire au sol. Les coordonnées servent uniquement au positionnement des cités et au calcul de distance/temps de trajet pour les troupes et commerçants.
* **Point stratégique central :** Un emplacement fixe aux coordonnées centrales de la carte accueille la **Merveille** (site neutre contesté par l'ensemble des joueurs).

### **Gestion des utilisateurs & Rôles**
* Séparation stricte des rôles : un utilisateur est soit **`JOUEUR`**, soit **`ADMIN`** (maître du jeu), jamais les deux :
  * **Joueur :** Crée son royaume, gère son économie et ses troupes, participe à la partie[cite: 3].
  * **Administrateur (Maître du jeu) :** Ne possède pas de royaume dans la partie. Il supervise la session, modère/bannit les comptes, consulte le classement, injecte manuellement des bonus/malus et déclenche des événements narratifs ou climatiques[cite: 3].

---

## **3. Démographie & Races**

### **Choix de la faction / race**
À la création de son royaume, le joueur choisit sa population parmi 5 archétypes[cite: 3]. Chaque race confère un **trait passif économique unique**, une **capacité spéciale**, un **bonus d'alliance/tactique** et une **troupe spéciale** :
* **Roi des Canards :** Spécialistes du surnombre et de la transe collective (défense et assaut de masse)[cite: 1, 3].
* **Roi des Teuteus :** Économie frugale (réduction drastique de la consommation de nourriture) et résilience[cite: 1, 3].
* **Roi des Goblins :** Harcèlement à distance (archerie, jet de projectiles) et pillage agressif[cite: 1, 3].
* **Princesse des Dragons :** Puissance destructive lourde (destruction ciblée de bâtiments ennemis, capacités de feu)[cite: 1, 3].
* **Reine Licorne de la Forêt Arc-en-Ciel :** Synergies d'alliance, génération magique de ressources (or) et soutien[cite: 1, 3].

### **Gestion active de la population**
La population n'est plus un simple compteur passif ; les citoyens sont une ressource active qui se répartit dynamiquement :
* **Population disponible :** Citoyens inactifs prêts à être mobilisés[cite: 3].
* **Population ouvrière :** Citoyens assignés aux chantiers de construction ou d'amélioration de bâtiments.
* **Population en entraînement :** Citoyens en caserne en cours de formation militaire.
* **Population militaire :** Soldats d'active stationnés en garnison ou envoyés en expédition.
* **Conversion démographique :** $1\text{ citoyen} \rightarrow 1\text{ soldat}$. Entraîner une troupe consomme obligatoirement 1 point de population civile disponible.
* **Croissance :** Augmentation naturelle quotidienne/périodique, plafonnée par le niveau des Maisons du royaume[cite: 3].

---

## **4. Bâtiments & Chantiers de construction**

Le développement matériel du royaume s'appuie sur une hiérarchie de bâtiments (patron Héritage). Chaque structure a un rôle dédié :

| Bâtiment | Rôle et impact en jeu |
| :--- | :--- |
| **`Chateau`** | Cœur du royaume. Son niveau définit le niveau global du joueur, débloque les paliers technologiques et constitue un axe majeur de victoire (niveau 10). Bâtiment très coûteux. |
| **`Maison`** | Augmente le plafond maximal de population supporté par le royaume. |
| **`Scierie`** | Produit et accélère la récolte de Bois. |
| **`Carriere`** | Produit et accélère l'extraction de Pierre. |
| **`Taverne`** | Produit la Nourriture nécessaire à la subsistance et à la fête. |
| **`MineDOr`** | Génère de l'Or en continu. Bâtiment vulnérable : une partie de l'or est directement pillée/détruite lors d'une défaite militaire défensive. |

### **Système de chantiers (`ChantierConstruction`)**
* Pour lancer la construction ou l'amélioration d'un bâtiment, un **seuil minimal d'habitants** doit obligatoirement être assigné au chantier.
* **Mécanique d'accélération :** Plus le joueur affecte d'ouvriers au-delà du seuil minimal, plus la durée réelle du chantier diminue.
* Une fois le chantier terminé, les ouvriers sont automatiquement libérés et réintégrés dans la population civile disponible.

---

## **5. Économie & Ressources**

### **Types de ressources**
Le système repose sur 4 ressources fondamentales :
* **Bois :** Matériau de base pour les bâtiments, engins et armes de trait[cite: 1].
* **Pierre :** Essentielle pour les fortifications, le Château et la Merveille.
* **Nourriture :** Entretien de la population et formation des troupes[cite: 1].
* **Or :** Monnaie d'échange, technologies avancées, troupes d'élite[cite: 1].

### **Paliers d'évolution (`PalierNiveau`)**
Le passage aux niveaux supérieurs (via le Château) requiert :
* Un stock minimum validé de chaque ressource[cite: 3].
* Un seuil plancher de population globale[cite: 3].

---

## **6. Commerce & Échanges**

Les échanges économiques entre joueurs empruntent deux canaux distincts :

1. **Le Marché Public (`<<singleton>> Marche`) :**
   * Plateforme d'annonces globale et publique (`OffreMarche`).
   * Tout joueur peut publier une offre de vente (`ressourcesOffertes` contre `ressourcesDemandees`).
   * Les ressources mises en vente sont débitées du royaume vendeur dès la publication et placées sous séquestre.
   * N'importe quel souverain solvable peut accepter l'offre pour conclure instantanément le transfert.
   * L'offreur peut retirer son annonce tant qu'elle n'est pas conclue (remboursement des ressources).
2. **Les Échanges Privés Directs (`PropositionEchange`) :**
   * Négociation bilatérale de gré à gré entre deux royaumes ciblés (statuts : `EN_ATTENTE`, `ACCEPTE`, `REFUSE`, `ANNULE`)[cite: 3].

---

## **7. Système Militaire, Conquête & Pillage**

### **Triangle de combat (Pierre-Feuille-Ciseaux)**
L'armée repose sur trois classes d'unités de base, complétées par les unités spéciales de chaque race :
* **Lanciers :** Très efficaces contre la Cavalerie, vulnérables face aux Archers.
* **Cavaliers :** Mobiles et puissants contre les Archers, écrasés par les Lanciers.
* **Archers :** Attaquent à distance les Lanciers, fauchés par les charges de Cavalerie.

### **Progression martiale**
* Chaque type de troupe possède un **niveau d'amélioration** (recherche militaire) qui accroît ses statistiques de base (points de vie, attaque, vitesse de déplacement).
* Les troupes sont soit en faction défensive (**Garnison**), soit déployées en mission extérieure (**Expédition / Attaque**) vers des coordonnées cibles.

### **Capacité spéciale de défense : Le Charme**
Certains souverains disposent d'un pouvoir de charme : lors d'un assaut ennemi sur leurs remparts, un pourcentage des assaillants peut être séduit et changer de camp pour rejoindre gratuitement les rangs de la garnison locale.

### **Défaite militaire, Ruine & Renaissance**
* **Royaume abandonné :** Un souverain anéanti ou inactif laisse derrière lui une cité abandonnée qui reste présente sur la carte et peut être continuellement pillée par les voisins.
* **Renaissance :** Si un joueur perd l'intégralité de sa population ou subit une défaite totale, il a l'option de **redémarrer un nouveau royaume immédiatement**, au même emplacement cartographique.

---

## **8. Conditions de victoire (Win conditions)**

Une partie prend fin et sacre un vainqueur selon l'une des trois conditions suivantes :

1. **Course au Prestige (Le Roi Bâtisseur) :**
   * Élever son Château au **niveau 10**.
   * Dès qu'un joueur entame ou atteint ce palier, une alerte générale retentit sur toute la carte. Les autres joueurs disposent d'une fenêtre de temps pour former une coalition et attaquer en masse afin de détruire le Château avant la victoire définitive.
2. **Contrôle de la Merveille (Domination Centrale) :**
   * Sécuriser la zone centrale de la carte et y déployer ouvriers et garnison pour construire la **Merveille**.
   * Le joueur (ou l'alliance) qui achève l'édifice et le défend pendant le temps réglementaire remporte la partie.
3. **Survie & Domination Totale :**
   * Être le dernier royaume encore debout suite à l'élimination ou l'abandon de l'intégralité des 9 autres souverains.

---

## **9. Événements aléatoires & Maître du jeu**

* **Déclenchement automatique :** Fléaux périodiques (famine, invasion barbare, épidémie) ou bénédictions (moisson abondante, foire marchande)[cite: 3].
* **Intervention Admin :** L'administrateur peut introduire des perturbations en temps réel pour relancer la dynamique d'une partie (tempêtes de feu, trésors cachés)[cite: 3].
* **Événements à choix :** Les événements imposent parfois un dilemme moral ou stratégique au joueur, modifiant ses réserves ou son bonheur intérieur[cite: 3].