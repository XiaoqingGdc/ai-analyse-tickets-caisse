# ai-analyse-tickets-caisse

Comparer les prix de mes supermarchés habituels autour de Nantes (E.Leclerc, Super U, Carrefour) à partir de mes propres tickets de caisse, et contribuer ces prix à la base ouverte [Open Prices](https://prices.openfoodfacts.org) (Open Food Facts).

## Pourquoi ce projet

Open Prices ne contient quasiment aucun prix pour mes magasins (1 seul prix sur tout Carquefou en octobre 2026). Plutôt que d'analyser des données existantes, ce projet **produit sa propre donnée** : collecte des tickets, extraction automatique, contrôle qualité, puis analyse.

Questions visées :
- Quel magasin est le moins cher pour **mon** panier réel ?
- Le classement change-t-il une fois intégrés les promotions et avantages carte de fidélité ?
- Comment les prix évoluent-ils semaine après semaine ?

## Pipeline

```
tickets PDF ──► extraction OCR ──► contrôles qualité ──► CSV ──► SQL ──► Power BI
(data/raw)      (src/)                                  (data/processed)
```

`src/extract_tickets.py` traite les tickets E.Leclerc dématérialisés (PDF contenant une image) :
1. extraction de l'image du ticket (PyMuPDF)
2. prétraitement (recadrage, agrandissement, binarisation)
3. OCR Tesseract en 3 variantes, puis **vote majoritaire** ligne par ligne
4. contrôles : quantité × prix unitaire = total de ligne, somme des lignes = total du ticket, nombre d'articles annoncé

Sorties :
- `data/processed/ticket_lines.csv` : une ligne par produit (magasin, date, rayon, libellé, quantité, prix unitaire, total)
- `data/processed/ticket_discounts.csv` : remises immédiates (lots, 2+1…)

## Installation

```bash
pip install -r requirements.txt
```

Tesseract OCR avec le pack de langue français :
- Windows : installeur [UB Mannheim](https://github.com/UB-Mannheim/tesseract/wiki) (cocher *French* dans « Additional language data »), puis si besoin
  `set TESSERACT_CMD=C:\Program Files\Tesseract-OCR\tesseract.exe`
- Alternative : déposer `fra.traineddata` dans un dossier `tessdata/` à la racine du projet

## Utilisation

```bash
python src/extract_tickets.py data/raw/tickets/*.pdf
```

Chaque ticket est marqué `[OK]` ou `[A VERIFIER]` avec le détail des incohérences.

## Données et vie privée

Les tickets bruts (`data/raw/`) ne sont **pas** versionnés : ils contiennent des numéros de carte cadeau, codes d'autorisation et soldes fidélité. Seules les données produits/prix sont utilisées.

## Avancement

- [x] Extraction des tickets E.Leclerc
- [ ] Tickets Super U et Carrefour
- [ ] Correspondance libellés ticket → produits (codes-barres Open Food Facts)
- [ ] Contribution des prix à Open Prices
- [ ] Modèle SQL + tableau de bord Power BI
