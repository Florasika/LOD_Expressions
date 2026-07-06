# 🎯 Jour 4 / 10 — Tableau : LOD Expressions

> **Série : 10 Days of Tableau** · Jour 4/10  
> Concepts : FIXED · INCLUDE · EXCLUDE · Différence avec les agrégats classiques

<img width="1906" height="887" alt="Capture d&#39;écran 2026-07-06 151045" src="https://github.com/user-attachments/assets/448b139b-773a-41e8-892f-57eef92a6203" />
<img width="1901" height="840" alt="Capture d&#39;écran 2026-07-06 151059" src="https://github.com/user-attachments/assets/304e980e-4adf-49f0-8339-2393148efcb1" />
<img width="1908" height="872" alt="Capture d&#39;écran 2026-07-06 151112" src="https://github.com/user-attachments/assets/bb5841d8-e382-4176-b0d1-272097fc6f54" />
<img width="1651" height="781" alt="Capture d&#39;écran 2026-07-06 151125" src="https://github.com/user-attachments/assets/71c5ae16-b571-48a0-b430-b854229dfbbf" />

---

## 📁 Fichiers du projet

```
day-04-lod/
│
├── ventes_lod_tableau_j4.xlsx    ← 373 lignes · 40 clients · 12 mois · 3 segments
└── README.md
```

---

## 🚀 Connexion des données

```
1. Ouvrir Tableau
2. Fichier → Se connecter → Microsoft Excel → ventes_lod_tableau_j4.xlsx
3. Feuille "Ventes LOD" → glisser sur le canvas
4. Cliquer "Feuille 1"
```

---

## 🧠 C'est quoi une LOD Expression ?

**Le problème :** dans Tableau, les calculs s'agrègent selon le niveau de détail de la VUE. Si ta vue est au niveau Vendeur, `SUM([Montant])` donne le total par vendeur. Si tu passes en niveau Région, le même calcul change automatiquement.

Une **LOD (Level of Detail)** fige le niveau de calcul **indépendamment de la vue** — peu importe comment tu zoomes ou filtres.

```
Calcul classique : SUM([Montant])      → s'adapte à la vue
LOD FIXED        : {FIXED [Vendeur] : SUM([Montant])} → toujours par vendeur
```

---

## 🔑 FIXED — figer le calcul sur des dimensions précises

**FIXED** calcule à un niveau que tu définis, quels que soient les filtres de la vue.

### LOD 1 — CA Total par Vendeur (indépendant de la vue)

```
Feuille de calcul → Analyse → Créer un champ calculé

Nom : CA Total par Vendeur (LOD)
Formule :
    { FIXED [Vendeur] : SUM([Montant]) }
```

**Usage :** glisse ce champ sur une vue par Produit → tu vois quand même le CA total
du vendeur associé à chaque ligne, pas le CA de ce produit pour ce vendeur.

---

### LOD 2 — CA Total par Client (pour classer les clients)

```
Nom : CA Total Client
Formule :
    { FIXED [Client] : SUM([Montant]) }
```

**Usage :** afficher le CA cumulé de chaque client sur toute la période, même dans
une vue filtrée par mois ou par produit.

---

### LOD 3 — Première date d'achat par Client (date d'acquisition)

```
Nom : Première Commande Client
Formule :
    { FIXED [Client] : MIN([Date]) }
```

**Usage :** calculer l'ancienneté client, identifier les nouveaux clients d'un mois.

---

### LOD 4 — Nombre de transactions par Client

```
Nom : Nb Transactions Client
Formule :
    { FIXED [Client] : COUNTD([Date]) }
```

**Usage :** segmenter les clients par fréquence d'achat.

---

### LOD 5 — Segment de performance Vendeur

```
Nom : Performance Vendeur
Formule :
    IF { FIXED [Vendeur] : SUM([Montant]) } > 50000
    THEN "Top Vendeur"
    ELSE "Standard"
    END
```

**Usage :** colorer les barres d'un graphique selon la performance globale du vendeur,
même dans une vue filtrée sur un seul mois.

---

## 🔑 INCLUDE — ajouter une dimension au calcul

**INCLUDE** calcule à un niveau PLUS DÉTAILLÉ que la vue en ajoutant une dimension.

### LOD 6 — Moyenne des CA par Client dans chaque Région

```
Nom : Moy CA par Client par Région
Formule :
    { INCLUDE [Client] : AVG([Montant]) }
```

**Pourquoi :** si ta vue est au niveau Région, `AVG([Montant])` donne la moyenne
par transaction. INCLUDE [Client] calcule d'abord la somme par client, puis
fait la moyenne de ces sommes — résultat très différent.

---

### LOD 7 — Panier moyen par client (indépendant de la vue)

```
Nom : Panier Moyen Client
Formule :
    { INCLUDE [Client] : SUM([Montant]) / COUNTD([Date]) }
```

**Usage :** dans une vue par Vendeur, voir le panier moyen réel des clients
de chaque vendeur — et non la moyenne de toutes les transactions.

---

## 🔑 EXCLUDE — retirer une dimension du calcul

**EXCLUDE** calcule à un niveau MOINS DÉTAILLÉ que la vue en ignorant une dimension.

### LOD 8 — % du CA que représente chaque Vendeur sur sa Région

```
Nom : CA Région (sans Vendeur)
Formule :
    { EXCLUDE [Vendeur] : SUM([Montant]) }
```

Puis créer un second champ calculé :

```
Nom : Part du CA Région
Formule :
    SUM([Montant]) / [CA Région (sans Vendeur)]
```

Format : Pourcentage, 1 décimale.

**Usage :** dans une vue Vendeur × Région, voir la part de chaque vendeur dans
sa région sans créer un calcul de table (plus stable avec les filtres).

---

## 🚀 Construire les 3 vues

### Vue 1 — Classement Clients par CA Total (FIXED)

```
1. Créer le champ : CA Total Client = { FIXED [Client] : SUM([Montant]) }

2. Nouvelle feuille :
   Lignes    → Client
   Colonnes  → CA Total Client  (glisser le champ LOD)
   Trier     → décroissant
   Couleur   → Segment Client
   Filtre    → "Premiers" → Top 10 par CA Total Client
   
3. Renommer : "Top 10 Clients"
```

---

### Vue 2 — Nouveaux clients par mois (FIXED + MIN)

```
1. Créer : Première Commande = { FIXED [Client] : MIN([Date]) }

2. Créer : Est Nouveau Client
   Formule :
       IF MONTH([Première Commande]) = MONTH([Date])
       AND YEAR([Première Commande]) = YEAR([Date])
       THEN "Nouveau" ELSE "Récurrent" END

3. Nouvelle feuille :
   Colonnes  → Mois
   Lignes    → COUNT([Client])
   Couleur   → Est Nouveau Client
   Type      → Barres empilées
   
4. Renommer : "Nouveaux vs Récurrents"
```

---

### Vue 3 — Part de marché Vendeur par Région (EXCLUDE)

```
1. Créer : CA Région = { EXCLUDE [Vendeur] : SUM([Montant]) }
2. Créer : Part CA   = SUM([Montant]) / [CA Région]

3. Nouvelle feuille :
   Lignes    → Vendeur
   Colonnes  → Région
   Valeur    → Part CA (en texte/couleur)
   Type      → Carte thermique (Heat Map)
      → Repères → changer en "Carré"
      → Taille → agrandir
      → Couleur → Part CA (dégradé blanc → bleu)
      → Étiquette → Part CA (format %)
   
4. Renommer : "Part de marché par Région"
```

---

## 🧠 Récap — Quand utiliser quoi ?

| LOD | Quand l'utiliser |
|-----|-----------------|
| `FIXED` | Calculer indépendamment de la vue et des filtres de dimension |
| `INCLUDE` | Affiner le calcul en ajoutant une dimension absente de la vue |
| `EXCLUDE` | Calculer un total partiel en ignorant une dimension de la vue |

### Règle mémo rapide

```
FIXED   = "calcule toujours comme ça, peu importe la vue"
INCLUDE = "descends plus bas dans le détail"
EXCLUDE = "remonte d'un niveau de détail"
```

### ⚠️ Ordre des filtres et LOD

- `FIXED` **ignore** les filtres de dimensions (sauf filtres de contexte)
- `INCLUDE` et `EXCLUDE` **respectent** les filtres de dimensions
- Pour qu'un filtre s'applique à un FIXED → le convertir en filtre de contexte

---

## 💡 Erreurs courantes

| Erreur | Cause | Solution |
|--------|-------|---------|
| Résultat identique au SUM classique | Dimension déjà dans la vue | Essayer INCLUDE à la place |
| LOD ignoré par le filtre | FIXED ignore les filtres dim. | Ajouter le filtre au contexte |
| Agrégation impossible | LOD dans une mesure non agrégée | Entourer de SUM() : `SUM({ FIXED... })` |

---

---

⭐ **Si ce projet t'aide, mets une étoile !**
