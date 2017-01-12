## BALTIC: the Backronymed Adaptable Lightweight Tree Import Code

baltic was initially developed to extract various statistics from molecular phylogenies derived from [BEAST](https://github.com/beast-dev/beast-mcmc) in a customised way. My [influenza B virus reassortment paper](https://dx.doi.org/10.1093/molbev/msu287) used early versions of baltic’s code to look at how the human influenza B virus segment diversity is structured according to genomic background. I’ve since split up the various bits of code into three parts:

--------------------

## baltic
[`baltic.py`](baltic.py) is the tree parser itself. It uses three main classes - node, leaf and tree to import, manipulate and plot BEAST trees with their rich variety of comments. Node and leaf classes have references to the usual set of parameters you would find in a phylogeny - index of character in string designating the branch (a unique identifier within the tree), length, height, position in time (absoluteTime), X and Y coordinates, a dictionary encoding BEAST branch comments (traits) and a reference to their parent (None for root) and a string designating their type (branchType). The node class additionally have a children attribute, which is a list of the node’s children, another list called leaves that contains tip names that descend from that node, a numChildren attribute, which is the length of the leaves list and a childHeight attribute which tracks when the last tip descending from the node existed. The leaf class has two extra attributes called name and numName. Trees drawn from the posterior distribution will usually encode tips as numbers to save space and require a translation map to convert back into actual interpretable tip names. In baltic numName will be the exact name for the tip that was used in the tree string, with functions to allow the translation of numName into name.

baltic.py evolved from a [short linked list script on StackOverflow](http://stackoverflow.com/questions/280243/python-linked-list/280286#280286) and underwent a major overhaul in order to correct [an article](https://dx.doi.org/10.1126/science.aaa5646) that was wrong. The code should be fairly legible (and commented) and highly adaptable to suit anyone’s needs.

--------------------
## Usage basics

By convention baltic is imported as bt:

`import baltic as bt`

For every tree that we want to import we need an empty tree object:

`myTree = bt.tree()`

Then call the `make_tree()` function with the tree string and the empty container:
```python
treeString='((A:1.0,B:2.0):1.0,C:3.0);'

bt.make_tree(treeString, myTree)
```
Note it means that you'll have to write some code to parse out the tree string if you're not dealing with a newick file. baltic will warn the user if it can't parse something. If this happens you should check if your tip names or annotations contain characters that should **never** be found outside of functional tree string bits, such as commas or parentheses. Alternatively, it may be that the regexes that are used to parse out tip names or annotations don't cover some special character you use to define your taxa and will require some editing of baltic.py to alleviate the problem. Feel free to raise an issue if this happens.

`make_tree()` is a function that parses the tree string and interacts with the tree class to build the data structure that is the phylogenetic tree. It works in the following way:

- Every time an opening parenthesis (`(`) is encountered in the tree string a new instance of `node` class is created. The new class' `.index` attribute is set to the index along the tree string where it was encountered, giving that particular class a unique identifier within the tree string. The `.parent` attribute is set to whatever the previous object encountered was, similarly, since the last encountered object could only be another node, the current node is added to its parents list of children. Finally we set our new node as the 'current' node of the tree and append the node to the list of objects (`.Objects`, which are branches) contained in the tree.

- Every time a string is encountered which may or may not be surrounded by quotation marks (`'` or `"`) or have the beginning of an annotation block (`[`) we create a new `leaf` class. It also receives an `.index` identifier, like the `node` class. Unlike the `node` class, however, the `.numName` attribute is also set as the string that defined the tip. In BEAST trees it will be the number that identifies the tip, but it could also be a regular string.

- Next baltic looks for annotations, which are the blocks in the format `[&parameter1=1.0,parameter2=0.0]`. These are transformed into the `.traits` dictionary for the branch. In this example the branch being parsed would receive a dictionary with two keys: `cur_branch.traits['parameter1']=1.0` and `cur_branch.traits['parameter2']=0.0`.

- Annotations should be followed by branch lengths preceded by a colon (`:`). The branch length is assigned to the current branch's `.length` attribute.

- Forks in the tree string are defined as commas (`,`) and ends of clades are defined by closing parentheses (`)`) and both mean that whatever comes next is in relation to the parent branch of whatever branch we were dealing with earlier.

- Finally tree strings are finished with a semi colon (`;`).


Before you can run any analysis you will usually have to traverse the tree such that branch lengths which are available in the tree string are transformed into branch heights:

`myTree.traverse_tree()`

This takes the `.length` attributes of each branch and sums or subtracts them, as appropriate during a tree traversal and the `.height` attribute of each branch (`node` or `leaf` object) is set, where the root of the tree has `.height = 0.0` and the most recent tip is the highest object in the tree. The tree traversal will also set the tree's `.treeHeight` attribute.

If your tree happens to have branch lengths in units of time you can use the `.height` attribute to modify the `.absoluteTime` attribute of each branch, such that the entire tree is calibrated and position correctly in time. This involves finding the most recent tip in the tree (in absolute time), subtracting the `.treeHeight` and adding the `.height` of the each branch.


Most analytic operations will involve looking at each branch individually without referring back to the tree structure much, beyond immediate children or parents of a particular branch. This is done by iterating over the tree's `.Objects` list, which contains all the branches in the tree. If you want to print out the height of each internal branch whose parent had a different trait value you would do it as:

```python
for k in myTree.Objects:
   if isinstance(k,bt.node) ## (or, alternatively if k.branchType=='node')
       if k.traits[myTrait] != k.parent.traits[myTrait]:
           print k.height
```

--------------------

## samogitia
![](figures/coa_samogitia.jpg)

[`samogitia.py`](samogitia.py) is the heavy-lifting, tree file-wrangling script in the collection. It’s main role is to parse BEAST tree files, use baltic to create tree data structures, which samogitia then manipulates to create BEAST-like log files that can usually be imported into [Tracer](http://tree.bio.ed.ac.uk/software/tracer/) or used in another program.

--------------------

## austechia
![](figures/coa_austechia.png)

[`austechia.ipynb`](austechia.ipynb) is the fancy Jupyter notebook that takes tree files, usually MCC trees from BEAST, and plots them. It is meant to be part teaching tool to get people to think about how trees are plotted, to allow for highly customisable representations of trees (e.g. Fig 6 in my [MERS-CoV paper](http://dx.doi.org/10.1093/ve/vev023)) and to improve the aesthetics situation in phylogenetics.

--------------------

Copyright 2016 [Gytis Dudas](https://twitter.com/evogytis). Licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).
