# Template for Network-Based Career Mapping Visualization
Have you ever wanted to know what your next few positions within a company should be based on your current skills and the differences in salary between positions? Did you ever want to increase retention and foster prolonged development amongst your workforce? Well, I certainly wanted to do both, and I wrote this visualization algorithm as a preliminary step / PoC in deploying such a solution. 

## Results
![Drag Racing](https://github.com/Fehiroh/Network-Visualization-Work/blob/main/position_skillset_similarity.jpg)
Explanation: 

## Initial Imports
We're going to be using itertools to generate edges in a computationally efficient manner (we only have nine nodes in this example, but I code for scalability out of habit), random in order to randomize the inputs while maintaining reproducability, and Matplotlib for visualization. Funnily enough, despite lines playing a critical role in almost all network graphs, Line2D only gets used to draw the legend.  
```
import itertools
import random
import matplotlib.pyplot as plt
from matplotlib.lines import Line2D 
```
## Creating the Initial Seed and Nodes
In order to create a randomized (but reproducable) network graph, we will create nine nodes, and set the seed for randomization to '4242' in honour of Douglas Adams. 
```
series = "123456789"
nodes = list(series)
random.seed(4242)
```

# Setting up the Nodes and Edges 
Because this project is centered on creating a template for career map visualization, it makes sense that one node must be the current position an Employee has within the company. We randomly select that node. In an actual deployment, the position of a given employee would be either be fed into the visualation function as a parameter, or the deployment would be interactive, in which case there would be a dropdown with all positions within the company.  
```
# Set a random node to be the position an employee currently occupies
current_position = nodes[random.randint(0,len(nodes)-1)]
```
After determining the starting position, we generate the salaries of each position. Most career moves are done with compensation in mind, so we are going to use these to direct the edges of the graph so that the arrows point towards positions that have higher compensation. Going for higher compensation with a position change isn't always the case, but it adds readability and creates a clearer "path". Practically, we only add directional edges from a lower paying position to a higher paying one.  

```
#Create fake salaries to drive Directionality of Graph
node_salaries = {}
for i in range(len(nodes)):
    node_salaries[nodes[i]] = random.randint(1,100)/100

# Create edges for each possible connection, based on values in node salary     
combos = [x for x in itertools.permutations(series, 2) 
          if node_salaries[x[0]] < node_salaries[x[1]]]
```

Similarly, the next step is to determine which nodes have a lower salary than the position an employee currently holds. This list is used later in order to distinguish careers that would be a step-down for the employee so that they can focus on where they are likely to go. We don't remove these entirely, as they can sometimes form stepping stones to higher paying positions that are too disimilar (skillwise) to the starting position to be a tenable next position. This maintains a nice balance between making the graph easy to read for those focused on immediate and obvious moves while allowing employees to consider less traditional paths. 

```   
lower_salaries_than_starting = [x for x  in nodes if node_salaries[x] < node_salaries[current_position]]
```

Having mentioned skillset discrepancies between positions, it's now time to generate a random value for each edges that falls between 0 and 1. In a realworld situation, this can be generated a number of ways. My personal favorite is a three step solution. Firstly, you'd need to create a company-wide skill inventory, where every skill for every position is listed and levels of proficiency are described in concrete terms. Next, Hiring Managers and Talent Acquisition would collaborate in order to assign a score for each skill required to hold each position.  In the cases where a given skill isn't needed (JavaScript for Custodial staff, for instance), you simply assign the position a 0. Next, you can treat these skills as spatial dimensions and measure the distance between positions then scale the positions to fall between 0 and 1. 
```
# Generate  Similarity of Positions [ie, nodes]
combo_relationships = {}
for i in range(len(combos)):
    combo_relationships[combos[i]] = {"weight" : (random.randint(1, 100)/100)}
```

    
# The Network
Okay, after a lot of setup, the graph object can finally be created and we can begin to populate it with nodes and edges. We note which nodes are not the current position, add our edges that show skillset similarity and salary differential, and then create several edgesets. These are tuples within a list that we will use to visually differentiate similar positions from disimilar positions.  We also organize the nodes according to their similarity using 600 iterations of the Fruchterman-Reingold force-directed algorithm.
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
## Plotting the Network / Career Map
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

# Output: 
![Drag Racing](https://github.com/Fehiroh/Network-Visualization-Work/blob/main/position_skillset_similarity.jpg)
