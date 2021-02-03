---
title: networkx基础
date: 2019-09-10 11:36:10
tags:
- networkx
categories: graph
---

## 建图

最简单的undirected graph:

```python
import networkx as nx
g = nx.Graph
```

### add nodes

可以通过一次加一个，也可以多个

- 一次加一个，**这样方便一起加入attributes（node或者edge的）**，然后再在外面套for

  ```python
  for node, value in zip(split_sorted_indices.tolist(),
                         zip(split_sorted_node_features, split_sorted_eval_label.tolist(), split_sorted_loss.tolist(),
                             split_original_label.tolist())):
      attr_dict = dict(zip(['node_features', 'eval_label', 'loss', 'original_label'], value))
      g.add_node(node, **attr_dict)
  ```

 - 还可以一次加一组，但是这样attr所有node都一样了

   ```
   g.add_nodes_from([1, 2, 3,], attr={'a', 'b', 'c'})
   ```

   

### add edges

与nodes 同理，也有两种



## 单独对nodes/edges设features

### set_node_attributes

- 默认要求要传入一个**dict**， 并且这个dict要**以node为key，其value是对应node的attribute**。 如果传入的是一个list ()，则会！！**默认添加到每一个node上面，作为每一个node的attributes，并且更新这个list会影响到每一个node的attributes**

  

### set_edge_attributes

- 同理也是这样，**要求传入的是dict**







## Undirected Graph

1. nx.Graph就是undirected graph的行为，如果要directed graph 用 nx.DiGraph
2. 当node中没有所添加edge的node 时候，会自动添加那个node，**不会报错！！**
3. 一条edge的同一个node调换顺序添加两次，实质上加了一条，**即有自动去重功能！！**

```python
g.add_edge(1,0)
g.edges()
Out[30]: EdgeView([(1, 0)])
g.add_edge(0,1)
g.edges()
Out[32]: EdgeView([(1, 0)])
```



## 常用功能 

### 获取features

**用get_node_features/get_edge_features**

```python
nx.get_node_attributes(g_obj.nx_g, 'eval_label')
```

这样获得的是一个dict，因此还要为了转化为list，还要如下：

```python
list(nx.get_node_attributes(g_obj.nx_g, 'eval_label').values())
```



### 获取Subgraph

直接用`Graph.subgraph(nbunch)`即可，其中`nbunch`用一个node idx组成的list即可，会自动分离出subgraph。

- 默认attributes是mapping，所以对subgraph attributes的修改也会影响parent graph。欲没有影响，则应该用`Graph.subgraph().copy`
- 若是为了直接变小图，则应该用`G.remove_nodes_from([ n in G if n not in set(nbunch)])`



### 对nodes重新标注relabeling nodes

- 作用：对原来的nodes 做一个mapping，并顺带修改edges

### `nx.relabel_nodes`按任意mapping对nodes进行重新标注

[networkx.relabel.relabel_nodes](https://networkx.github.io/documentation/networkx-1.10/reference/generated/networkx.relabel.relabel_nodes.html#networkx.relabel.relabel_nodes)

- 只有`dict`中涉及到的nodes会进行relabel，其余的不会，**因此可以通过这个做部分mapping**

- 默认 **不是inplace操作，而是返回一个copy**,copy为false的时候为inplace

- 例子：把原来不是连续的120个nodes映射为从0开始的120的nodes

  ```python
  mapping = {idx:i for idx, i in zip(list(self.nx_g.nodes()), range(len(self.nx_g)))}
  t1 = nx.relabel_nodes(self.nx_g, mapping)
  t1.nodes()
  ```

  ```
  Out[5]: NodeView((0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119))
  ```

  



### `nx.convert_node_labels_to_integers`把原来的nodes映射成按一定规则的连续整数

- **默认是return a copy（包括nodes 和 edge attribute）**

- `first_label`设定mapping的起始点。
- `ordering`默认是按照原来的顺序进行映射，也可设定为`sorted`（按原来nodes排序后的顺序映射）。。。
- `label_attribute`设定了之后可以把 **原来的index放到新的graph的nodes attributes中**

- 例子：把原来不是连续的120个nodes映射为从0开始的120的nodes

  ```python
  relaled_nx_g = nx.convert_node_labels_to_integers(self.nx_g, label_attribute='original_index')
  relaled_nx_g.nodes.data()
  ```

  ```
  NodeDataView({0: {'original_index': 33633}, 1: {'original_index': 9209}, 2: {'original_index': 9409}, 3: {'original_index': 17208}, 4: {'original_index': 49102}, 5: {'original_index': 38536}, 6: {'original_index': 48712}, 7: {'original_index': 21370}, 8: {'original_index': 12408}, 9: {'original_index': 15}, 10: {'original_index': 42211}, 11: {'original_index': 20989}, 12: {'original_index': 30451}, 13: {'original_index': 37822}, 14: {'original_index': 3538}, 15: {'original_index': 25452}, 16: {'original_index': 21484}, 17: {'original_index': 26341}, 18: {'original_index': 44812}, 19: {'original_index': 3568}, 20: {'original_index': 9909}, 21: {'original_index': 38385}, 22: {'original_index': 18793}, 23: {'original_index': 45334}, 24: {'original_index': 4261}, 25: {'original_index': 23962}, 26: {'original_index': 34019}, 27: {'original_index': 34865}, 28: {'original_index': 4188}, 29: {'original_index': 35290}, 30: {'original_index': 44758}, 31: {'original_index': 45856}, 32: {'original_index': 17523}, 33: {'original_index': 7029}, 34: {'original_index': 30822}, 35: {'original_index': 10581}, 36: {'original_index': 45085}, 37: {'original_index': 39879}, 38: {'original_index': 20956}, 39: {'original_index': 745}, 40: {'original_index': 28055}, 41: {'original_index': 9002}, 42: {'original_index': 20605}, 43: {'original_index': 5760}, 44: {'original_index': 44025}, 45: {'original_index': 40070}, 46: {'original_index': 9452}, 47: {'original_index': 36719}, 48: {'original_index': 21872}, 49: {'original_index': 21308}, 50: {'original_index': 21437}, 51: {'original_index': 43565}, 52: {'original_index': 34192}, 53: {'original_index': 29074}, 54: {'original_index': 49724}, 55: {'original_index': 4775}, 56: {'original_index': 12937}, 57: {'original_index': 31258}, 58: {'original_index': 23364}, 59: {'original_index': 22012}, 60: {'original_index': 2209}, 61: {'original_index': 11265}, 62: {'original_index': 4642}, 63: {'original_index': 34976}, 64: {'original_index': 48237}, 65: {'original_index': 41181}, 66: {'original_index': 31586}, 67: {'original_index': 7856}, 68: {'original_index': 12218}, 69: {'original_index': 22005}, 70: {'original_index': 33008}, 71: {'original_index': 19581}, 72: {'original_index': 36933}, 73: {'original_index': 39880}, 74: {'original_index': 47584}, 75: {'original_index': 48416}, 76: {'original_index': 33893}, 77: {'original_index': 41449}, 78: {'original_index': 24186}, 79: {'original_index': 40210}, 80: {'original_index': 5433}, 81: {'original_index': 25687}, 82: {'original_index': 18562}, 83: {'original_index': 17578}, 84: {'original_index': 16795}, 85: {'original_index': 22099}, 86: {'original_index': 34094}, 87: {'original_index': 34325}, 88: {'original_index': 37083}, 89: {'original_index': 23797}, 90: {'original_index': 7596}, 91: {'original_index': 23219}, 92: {'original_index': 46978}, 93: {'original_index': 13869}, 94: {'original_index': 3817}, 95: {'original_index': 6197}, 96: {'original_index': 41453}, 97: {'original_index': 5826}, 98: {'original_index': 2925}, 99: {'original_index': 30976}, 100: {'original_index': 13219}, 101: {'original_index': 36579}, 102: {'original_index': 44174}, 103: {'original_index': 5899}, 104: {'original_index': 40043}, 105: {'original_index': 16236}, 106: {'original_index': 49027}, 107: {'original_index': 16925}, 108: {'original_index': 31563}, 109: {'original_index': 25800}, 110: {'original_index': 35287}, 111: {'original_index': 16076}, 112: {'original_index': 20097}, 113: {'original_index': 9579}, 114: {'original_index': 8929}, 115: {'original_index': 41201}, 116: {'original_index': 48798}, 117: {'original_index': 22029}, 118: {'original_index': 23635}, 119: {'original_index': 40777}})
  ```

  