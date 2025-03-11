## Node management

Similar to HTML, the entire HTML is composed of various nodes. The root node is the HTML tag, and then there are the child nodes, and then the child nodes nest the child nodes to form a node tree. Then the browser renders based on this node tree, and then The page we need is rendered, and Laya is actually the same. Different components and scenes form a node tree, which is rendered by the Laya engine.

Nodes are the basis of project development. Only by fully understanding how to operate nodes can complex requirements be realized faster. Each node component class inherits from the Node class, which is the base class for all objects that can be placed in the display list. This display list manages all objects displayed in the Laya runtime. Use the Node class to arrange display objects in a display list. Node objects can have child display objects.

Let’s get familiar with all the basic functions of the Node class

### Add to

- #### Add child node addChild(node:Node)


Without considering the hierarchy, we can add child nodes to a node, so that the child nodes can follow the parent node to display or move, and facilitate management. The added child nodes will be added to the end of all child nodes.

- #### Add a child node to the specified index position addChildAt(index:number)


It is often necessary to consider the hierarchical relationship or occlusion relationship of nodes, and if you already know that you need to add it to the specified hierarchical position, you can use this method.

- #### Add child nodes in batches addChildren(...args:any[])


Directly adding a group of child nodes to the parent node in order is equivalent to calling the addChild child node once in a loop.



### Find

- #### Get the child node object according to the name getChildByName(name: string)


When we know the name of a node, we can use this method to find a child node one level below the parent node.

- #### Get the child node object according to the index position of the child node getChildAt(index:number)


When we know the index position of a node, we can use this method to find a child node one level below the parent node.

- #### Get the index position of the child node according to the child node object getChildIndex(node:Node)


When we want to know the index position of a node, we can use this method to get the child nodes

- #### Get the number of child node objects numChildren


By calling the numChildren attribute of the parent node, you can get the number of all child nodes at the next level of the parent node, which can be used to traverse all child nodes. Note that the child nodes under the child node are not included.

- #### Get the parent node object parent


You can directly obtain the parent node by calling the parent attribute of the child node.

- #### Whether the current container contains the specified node object contains(node: Node)


To determine whether a node contains a child node, you can use this method. Note that this search process will traverse all nested child nodes.

- #### Check whether this node is the upper node of a certain node isAncestorOf(node: Node)




### Change

- #### Set the index position of the child node setChildIndex(node:Node,index:number)


If you want to change the display level of child nodes, you can call this method. Note that if the index exceeds the number of nodes of the parent node, an exception will be thrown.

- #### Pass in the new node and replace it at the index position of the existing child node replaceChild(newNode: Node, oldNode: Node)


If you want to replace an existing node with a new node, you can call this method. Note that the original child node will be removed but will not be destroyed.



### Remove

- #### Delete child node removeChild(node: Node)


This method is called when deleting a child node. Note that the child node will be removed but will not be destroyed.

- #### Remove yourself from the parent container removeSelf()


When you do not need to know the parent node, you can directly delete yourself from the parent node by calling this method. This method is used more frequently.

- #### Delete the corresponding child node according to name removeChildByName(name: string)


When we know the name of a node, we can use this method to delete the child node from the parent node.

- #### Delete the corresponding child node according to the index position removeChildAt(index: number)


When we know the index position of a node, we can use this method to delete the child node from the parent node

- #### Delete all child nodes in the specified index range removeChildren(beginIndex: number = 0, endIndex: number = 0x7fffffff)


This method can quickly delete all child nodes in the specified index range

