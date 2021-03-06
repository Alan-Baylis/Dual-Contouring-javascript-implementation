# Dual-Contouring-javascript-implementation


<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/octree3.png" width="450px">

This javascript implementation takes [Nick's excellent explanations](http://ngildea.blogspot.it/2014/11/implementing-dual-contouring.html) on the inner workings of the algorithm and expands it further in the topics he didn't discuss in depth, such as ContourCellProc's routines and the global variables of the original octree structure  of Tao Ju.

Since a blog post can't be a book, and considering the sheer amount of explanations required to fully grasp the algorithm, he omitted an in depth explanations on the inner isosurface extraction routines which will be discussed here along with a simple implementation of the algorithm in javascript togheter with a raw WebGL renderer to output the extraction results.

We'll define the following order convention for the children of each internal node:

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/childrenorder.png" width="200px">

Similiarly, the vertex order convention is as follow

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/vertorder.png" width="200px">

Starting from ContourCellProc(...),
This function acts on internal nodes only, and starts by recursively calling itself on each of its eight children

```javascript
...
for (i = 0; i < 8; i++) {
    ContourCellProc(node.children[i], indexBuffer);
}
...
```

It laters calls ContourFaceProc(...); on the 12 faces adjacent to the children of the current node, highlighted in gray in the following picture

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/ccpfaces.png" width="250px">

And finishes off by calling ContourEdgeProc(...) on the 6 shared edges depicted in red in the next picture

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/ccpedges.png" width="250px">


ContourFaceProc(...) is a bit more involved and it starts by checking if the provided nodes are actually internal, if they are it recursively
calls itself on the four faces shared by the adjacent children of both nodes. Assuming this function is called with the direction argument set to 0 (horizontal direction), 
it would try to recursively call it self on the four faces visible in the next image

The dark gray region represents the 4 shared faces, the light gray region encapsulate the children sharing those faces 

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/fpf.jpg" width="250px">

Afterwards this function will call ContourEdgeProc(...) on the four edges shared by the children of both internal nodes, highlighted here in red

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/fpe.jpg" width="250px">

ContourEdgeProc(...) needs 4 nodes with a common edge, if we consider the first of the four edges of the above image the selected children to pass over ContourEdgeProc(...) will be
those highlighted in gray in the next picture

<img src="https://github.com/Domenicobrz/Dual-Contouring-javascript-implementation/blob/master/screenshots/fpe2.png" width="250px">

Lastly, we're going to analyze ContourEdgeProc(...) and ContourProcessEdge(...)

ContourEdgeProc(...) is pretty straightforward

If the 4 nodes passed to this function are all leaves, they're just sent to ContourProcessEdge along with the direction identifier

If however, those 4 nodes are internal, we'll iterate on the 2 common edges shared by all of them, the next image shows one of those 2 edges 
and the associated 4 child nodes that would be passed to ContourProcessEdge(...)