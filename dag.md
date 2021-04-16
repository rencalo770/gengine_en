## Description
In order to increase support for any possible execution mode, and in order to allow users to arbitrarily arrange the order of execution of the rules, gengine supports the DAG (Directed acyclic graph, directed acyclic graph) execution mode. In this mode, gengine does not consider rules The specific priority is to execute the rules concurrently or sequentially based on the directed acyclic graph of the rules constructed by the user.

## graph storage
There are two main types of graph storage structures, the adjacency matrix storage method and the adjacency linked list method, and other storage methods are optimized on these two.
### Adjacency Matrix
The adjacency matrix is a matrix that represents the adjacency relationship between vertices, as follows

![](https://img-blog.csdnimg.cn/20190128175511539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMTQyMw==,size_16,color_FFFFFF,t_70)

### Adjacent Linked List
The graph's adjacency list storage method is a storage method that combines sequential allocation and chain allocation

![](https://img-blog.csdn.net/20180312142243881?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva29uZ194eg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## An improvement to image storage
If you directly let users directly construct the above two DAG storage structures to execute the rules, it is obviously a bit complicated. Therefore, in order for users to construct their own rule DAG more simply, we have made a little improvement on the adjacency matrix of the graph:** * The directed acyclic graph is abstracted into "layers", "layers" are arranged in columns, from left to right, the rules within the layers are executed concurrently, and the rules between layers are executed in sequence***.

In the following figure, a directed acyclic graph has 4 layers in total

![](https://upload-images.jianshu.io/upload_images/24932698-54d0b15d12e5b33b.png?imageMogr2/auto-orient/strip|imageView2/2/w/724/format/webp)

The user only needs to place the rule names of the same level in the same slice, and then use these slices as elements, sequentially from left to right, and place them in a slice in order to form a two-dimensional slice based on the rule name, and then call gengine For related APIs, gengine can execute rules based on the DAG graph constructed by the user.
As shown in the figure above, the rule engine will execute concurrently ["rule1","rule2","rule3"], and then concurrently execute ["rule4","rule5","rule6","rule7"], and then Execute ["rule8"], and finally execute ["rule9","rule10","rule11"] concurrently.

## Specific API

- in gengine.go  is ```func (g *Gengine) ExecuteDAGModel(rb *builder.RuleBuilder, dag [][]string) error```
- in gengine_pool.go  is ```func (gp *GenginePool) ExecuteDAGModel(dag [][]string, data map[string]interface{}) (error, map[string]interface{})  ```
- test: https://github.com/bilibili/gengine/blob/main/test/dag_test.go
