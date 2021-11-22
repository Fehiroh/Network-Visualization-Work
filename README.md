# Network-Visualization-Work
An algorithm I wrote as part of a project to link positions within a company via network graph as a means of promoting internal career pathing. 


## Results
![Drag Racing](https://github.com/Fehiroh/Network-Visualization-Work/blob/main/position_skillset_similarity.jpg)

## Initial Imports
```
import itertools
import random
import matplotlib.pyplot as plt
from matplotlib.lines import Line2D 
```
## Creating the initial 
```
series = "123456789"
nodes = list(series)
random.seed(4242)
```

# Label one position as current posttion 
```
current_position = nodes[random.randint(0,len(nodes)-1)]

#Create fake salaries to drive Directionality of Graph
node_salaries = {}
for i in range(len(nodes)):
    node_salaries[nodes[i]] = random.randint(1,100)/100

# Create edges for each possible connection, based on values in node salary     
combos = [x for x in itertools.permutations(series, 2) 
          if node_salaries[x[0]] < node_salaries[x[1]]]


lower_salaries_than_starting = [x for x  in nodes if node_salaries[x] < node_salaries[current_position]]

# Generate  similarity 
combo_relationships = {}
for i in range(len(combos)):
    combo_relationships[combos[i]] = {"weight" : (random.randint(1, 100)/100)}
```

    
# The Network
```
import networkx as nx
G = nx.DiGraph()

if current_position in nodes:
    nodes.remove(current_position)
    
non_current_positions = [x for x in nodes if x not in lower_salaries_than_starting]


# The Network - creating edgelists for coloration
for i in combo_relationships.keys():
    G.add_edge(i[0], i[1], weight = combo_relationships[i].get("weight"))

e_bad = [(u, v) for (u, v, d) in G.edges(data=True) if d["weight"] < 0.25]
e_okay = [(u, v) for (u, v, d) in G.edges(data=True) if d["weight"] >= 0.25 if d["weight"] < 0.5]
e_good = [(u, v) for (u, v, d) in G.edges(data=True) if d["weight"] >= 0.5 if d["weight"] < .75]
e_great = [(u, v) for (u, v, d) in G.edges(data=True) if d["weight"] >= 0.75]

pos = nx.spring_layout(G, iterations=600)

```
# Starting plotting
fig, ax = plt.subplots(1,figsize=(15,14))
ax = plt.gca()
ax.set_title('Comparing Positions: Skillset Similarity and Salary Differential',
              {'fontsize' : 25, "fontfamily": "serif"}, pad = 20, loc = "left")


# Rescale to honor the 1st rule of Geography
pos = nx.rescale_layout_dict(pos)


nx.draw_networkx_nodes(G, pos, nodelist = current_position, node_color = "black", label = "Current Position", node_size = 800, node_shape="o")
nx.draw_networkx_nodes(G, pos, nodelist = non_current_positions, label = "Other Positions", node_size = 500, node_shape = "d")
nx.draw_networkx_nodes(G, pos, nodelist = lower_salaries_than_starting, node_color = "grey", label = "Lower Pay", node_size = 600, node_shape = "p")

nx.draw_networkx_edges(
    G, pos, edgelist=e_bad, width=0.5, edge_color = 'r', alpha = 0.7, style = "dotted", 
    label = "0-24% Similarity",  arrowsize =12,  connectionstyle='arc3,rad=0.5', min_target_margin = 20, min_source_margin = 15
)
nx.draw_networkx_edges(
    G, pos, edgelist=e_okay, width=0.8, alpha=0.5, edge_color="black", style = "dashed",
    label = "25-49% Similarity",  arrowsize =12, connectionstyle='arc3,rad=0.3',  min_target_margin = 20,   min_source_margin = 15
)
nx.draw_networkx_edges(
    G, pos, edgelist=e_good, width=1, alpha=0.8, edge_color="black", 
    label = "50-74% Similarity",  arrowsize =12,  connectionstyle='arc3,rad=0.1',  min_target_margin = 20, min_source_margin = 15
)
nx.draw_networkx_edges(
    G, pos, edgelist =e_great, width=2, alpha = 0.9, edge_color ="g", 
    label = "75-100% Similarity", arrowsize =12, connectionstyle='arc3,rad=0.01',   min_target_margin = 20, min_source_margin = 15
)


nx.draw_networkx_labels(G, pos, font_color = "white", font_size = 14)



# Creating the legend 
custom_lines = [Line2D([0], [0], color="r", linestyle="--", lw=1, label="0-24% Skillset Similarity", alpha = 0.5),
                Line2D([0], [0], color="black", linestyle="--", lw=1, label="25-49% Skillset Similarity", alpha = 0.2),
                Line2D([0], [0], color="black", lw=1.2, label="50-74%  Skillset Similarity"), 
                Line2D([0], [0], color="green", lw=1.7, label="75-100% Skillset Similarity")]

ax.legend(loc ="lower left", handles=custom_lines, title = "Legend for Edges", title_fontsize = 'large')
fig.legend(loc="lower left", bbox_to_anchor=(0.13, 0.215, 0.5, 0.4), title= "Legend for Nodes", title_fontsize = 'large', 
          markerscale = 0.4)
plt.savefig('position_skillset_similarity.jpg')
```
