# CtlrC CtlrV

## První spuštění

```ps1
# 1. Vytvoření virtuálního prostředí
py -m venv .venv

# 2. Aktivace virtuálního prostředí
.venv\Scripts\activate

# 3. Nainstalování potřebných závislostí
py -m pip install -r requirements.txt

# 4. Spuštění aplikace Jupyter Notebook v adresáři s notebookem
jupyter notebook --notebook-dir=./notebook
```

## Importy a načtení .sav

```py
# == Importy ==
import itertools  # Vytvoření kombinací, pomůže při tvorbě pavučiňáku
import pathlib    # K souboru .sav se dostaneme relativní cestou

import hvplot.networkx as hvnx  # Pavučiňák
import networkx as nx           # Pavučiňák
import pandas as pd             # Práce s datovou maticí – datová analýza v Pythonu
import pyreadstat               # Čtení .sav souboru
from mlxtend.frequent_patterns import apriori, association_rules

pd.options.plotting.backend = "hvplot"  # Namísto matplotlib používáme hvplot + bokeh

# == Nalezení a načtení SPSS dat. Dostáváme pandas DataFrame a metadata. ==
PATH = pathlib.Path()
PATH_SHOPPING_ITEMS = PATH / ".." / "data" / "Shopping_items.sav"
df, meta = pyreadstat.read_sav(PATH_SHOPPING_ITEMS)
print(type(df))
# <class 'pandas.core.frame.DataFrame'>
print(type(meta))
# <class 'pyreadstat._readstat_parser.metadata_container'>
```

## Pavučinový diagram

```py
def web(nodes, edges, edge_colors):
    graph = nx.Graph()

    # = Uzly grafu =
    # Pro každý node přidáme jeden uzel do grafu a rozmístíme je na pomyslné kružnici (circular layout)
    graph.add_nodes_from(nodes)
    nodes_positions = nx.layout.circular_layout(graph)
    result_nodes = hvnx.draw_networkx_nodes(graph, nodes_positions, node_color="blue")

    # = Popisky uzlů =
    # Dokumentace zmiňuje nastavení fontu u popisků, bohužel tyto parametry nic nedělají (alespoň na mé straně)
    # Proto jsou kvůli čitelnosti popisky alespoň posunuty (v defaultu se nachází přímo na pozici uzlu, překrývá se)
    # (O kolik posunout)
    LABEL_OFFSET = 0.1
    # (Stejné souřadnice jako uzly, jen je druhá složka posunuta o LABEL_OFFSET)
    label_posistions = {k: [v[0], v[1] + LABEL_OFFSET] for k, v in nodes_positions.items()}
    result_labels = hvnx.draw_networkx_labels(graph, label_posistions)

    # = Hrany a jejich barvy =
    graph.add_edges_from(edges)
    result_edges = hvnx.draw_networkx_edges(graph, nodes_positions, edge_color=edge_colors, colorbar=True)

    return result_nodes, result_labels, result_edges
```

* Podpůrná funkce má parametry:
  * Uzly `nodes` – kolekce řetězců
  * Hrany `edges` – kolekce dvojic řetězců z `nodes`
  * Ohodnocení `edge_colors` – kolekce čísel, kde každé číslo z `edge_colors[𝑖]` odpovídá hraně z `edges[𝑖]`.
* Podpůrná funkce vrací trojici (uzly, popisky, hrany) pro vykreslení.

```py
# Podpůrná funkce nám vrátí iformace o uzlech, popiscích a hranách pro vykreslení
nodes, labels, edges = web(ITEM_TYPES, edges, edge_colors)

# Operace násobení zde slouží pro vykreslení všeho najednou
(nodes * labels * edges).opts(width=600, height=500)
```

## Evaluace – matice záměn doporučení alkoholu

```py
n_alcohol_recommended_correctly = 0        # True positive
n_alcohol_recommended_incorrectly = 0      # False positive
n_alcohol_not_recommended_correctly = 0    # True negative
n_alcohol_not_recommended_incorrectly = 0  # False negative

for _, row in flags.iterrows():
    # Row (pandas Series) bez sloupce s alkoholem je převeden na slovník stejně jako `in_my_cart_dict`
    row_cart = [k for k, v in row.drop(labels=["ALCOHOL"]).to_dict().items() if v == 1]
    # Doporučíme zákazníkovi alkohol?
    recommended_alcohol = recommend_alcohol(row_cart, a_rules_alcohol)
    # Koupil si zákazník skutečně alkohol?
    bought_alcohol = row["ALCOHOL"] == 1
    
    if recommended_alcohol:
        if bought_alcohol:
            n_alcohol_recommended_correctly += 1
        else:
            n_alcohol_recommended_incorrectly += 1
    else:
        if bought_alcohol:
            n_alcohol_not_recommended_incorrectly += 1
        else:
            n_alcohol_not_recommended_correctly += 1

# Matice záměn: sloupce jsou klasifikace, řádky jsou skutečnost
print("Alcohol\t  F\t  T")
print("     F \t", n_alcohol_not_recommended_correctly, "\t", n_alcohol_recommended_incorrectly)
print("     T \t", n_alcohol_not_recommended_incorrectly, "\t", n_alcohol_recommended_correctly)
```

## "User input"

```py
in_my_cart_dict = {
    'READMADE' : 0,
    'SNACKS'   : 0,
    'TOILETRY' : 0,
    'MILK'     : 0,
    'BAKERY'   : 0,
    'TINNED'   : 0,
    'FROZEN'   : 0,
    'ALCOHOL'  : 0,
    'VEG'      : 0,
    'MEAT'     : 0
}

in_my_cart = [k for k, v in in_my_cart_dict.items() if v == 1]
```
