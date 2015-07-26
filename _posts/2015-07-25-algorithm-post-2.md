---
layout: post
title: 二叉树的进化论(二)
---
*作为二叉树的新贵，2-3-4 tree让树群陷入了一片多结点的疯狂之中。*  
*树史中记载为结点大爆炸运动。*  
*树群们不断高喊造出5-node，6-node，甚至是100-node的口号，乐观的甚至已经在幻想一个结点统治世界的场景。*  
*实际上，按照这个趋势，树群们确实会发展出一种普遍的平衡树结构，B-tree。不过这也是以后的故事了。*
*多结点固然使高度的降低，天赋尽显。但凡事有利必有弊，对于插入和删除，代码的复杂度却大大地增加，尤其是超过4-node的结构。*  
*大爆炸运动使得理想的二叉树坚守者甚至认为这最终是向线性结构靠拢，破坏了二叉树的美感，于是联合起来，史称黑党，发动了反多大革命。*  
*一大批坚信大爆炸运动的树群也不示弱，立红党抗击。双方你来我往，但也说服不了对方，两党遂进入了长达几十个树年的对峙阶段。*
*就这样不知道过了多少年，一对红黑党的恋人偷偷结合，当时谁也没有想到，他们的种子在成长之后将是树群重新一统的树中霸王，红黑树。*

****

**Red-Black Tree**, 打破传统的二叉树新贵，胸怀树群大一统的野心。结构特点：  
    1. 结点是红色或者黑色的，简称**显色性**。  
    2. 根结点必然是黑色的，简称**根必黑**。  
    3. 叶子结点也是黑色的，简称**叶(也)黑**。  
    4. 一个结点是红色，则它的孩子结点必然是黑色，简称**红生黑**。由此也可推出，Red-Black Tree没有两个相邻的红结点，或者说，红结点的父子结点也都是黑结点。  
    5. 一个结点，其一直到后代叶子结点的所有简单路径包含相同数目的黑结点。简称**点叶一黑高**。黑高，指的是某结点到叶子结点的简单路径的黑结点数目。
所有维持这5项特性的二叉树,称为Red-Black Tree，简称RBT。

这RBT到底有什么神奇的地方呢？不就是多了个颜色么。  
没错，假如没有颜色属性，这RBT就是一棵二叉树而已。而正是这颜色的出现，魔法般地让RBT拥有了近乎2-3-4 tree一样低高度，高效率，还同时保留二叉树的美妙。  
那这到底是怎么做到的呢？    
答案还是在颜色身上，就是建立RBT和2-3-4 tree的联系。
  - 2-3-4 tree的2-node vs RBT的表示: 
  ![rbt-2node]({{"/img/rbt2node.png"}})  
  - 3-node vs RBT的表示:  
  ![rbt-3node]({{"/img/rbt3node.png"}})  
  - 4-node vs RBT的表示:  
  ![rbt-4node]({{"/img/rbt4node.png"}})  

题外话：有的书或者教材描述路径为红黑，本质上和结点没有区别。  

***
*一棵树是牛叉还是傻Ⅹ，都要靠查找，插入，删除来检验，这是树的命。 --《树经》*  

***  

决定树的查找，插入，删除的效率的关键因素便是树高了，如果我们能保证RBT的高度跟2-3-4 tree同等优秀，而且代码还更加清晰，简洁，那这就是一棵好树。  
为了方便接下来的表示，我们定义树的叶子作为哨兵，sentinel，表示为T.nil。  
根结点为**T.root**。并且使根结点的父亲结点为**T.nil**，那么RBT的结构将是这样的：  
![Red-Black Tree]({{"/img/RBT.png"}})

由此，我们知道所有的叶子结点都是不存储数据的，那么我们假设一棵RBT有N个内结点。  
为了方便表示，我们也把某个结点X的黑高(Black Height)，用BH(X)表示。  
对某结点X，N(X)表示以X为根的树的内结点数目。  
什么时候N可能最小呢？当RBT的结点都为黑，也就是退化为一棵二叉树的时候。  
![proof n]({{"/img/proofheight.png"}})  

上述证明RBT的高度不超过2lg(N+1)，那么可以保证查找，插入，删除操作的效率在O(lgN)。  
接下来，我们应该考虑，插入和删除的实现过程。  

RBT具有**显色性**，为了发挥红黑树的特性，插入的结点都假设为红结点。  
~~~~~~~~
{% highlight Java %}  
 //文中均是类Java伪代码
    Node insert(Node root, Key key, Value value) {
        if(root == T.nil) {
            return new Node(key, value, RED); //总是红结点
        }
        int cmp = key.compareTo(root.key);
        if(cmp < 0) {
            //key < root.key
            root.left = insert(root.left, key, value);
        }else if(cmp > 0) {
            //key > root.key
            root.right = insert(root.right, key, value);
        }else {
            //key == root.key
            root.value = value; //避免重复插入
        }
    ...
    }  

{% endhighlight %}  
~~~~~~~~

因此，插入存在两种结果：  
  1. 插入结点的父结点为黑结点，完成插入过程。  
  2. 插入结点的父结点为红结点，破坏了**红生黑**特性，那么就需要通过改变来维持特性。  
  具体的破坏情况又有四种：
  ![red-red]({{"/img/red-red.png"}})  

我们需要判断结点颜色： 
~~~~~~~~ 
{% highlight Java %} 
    //RED : true, BLACK : false
    static final boolean RED = true;
    static final boolean BLACK = false;
    boolean isRed(Node node) {
        if(node == T.nil) {
            return BLACK;
        }
        return node.color == RED;
    }
{% endhighlight %}   
~~~~~~~~

插入到3-node中，那么这几种情况需要经过转化才能得到4-node。  
![fix2red]({{"/img/fix2red.png"}})  
我们在插入后，需判断插入的新结点跟当前结点的关系，以及根据当前结点与父结点的关系来判断操作，我们引入一个lr_child表示当前结点是父结点的左孩子或右孩子。
~~~~~~~~
{% highlight Java %}  
    //lr_child, true-当前结点为左孩子，false-当前结点为右孩子
    Node insert(Node root, Key key, Value value, boolean lr_child) {
        ...
        if(cmp < 0) {
            //key < root.key
            root.left = insert(root.left, key, value, true);
            if(isRed(root) && isRed(root.left) && !lr_child) {
                    //若是上图right-left情况
                    //Rotate Right操作
                    root = rotateRight(root); 
            }
            if(isRed(root.left) && isRed(root.left.left)) {
                root = rotateRight(root);
            }
        }else if(cmp > 0) {
            //key > root.key
            root.right = insert(root.right, key, value, false);
            if(isRed(root) && isRed(root.Right) && lr_child) {
                    //若是上图left-right情况
                    //Rotate Left操作
                    root = rotateLeft(root); 
            }
            if(isRed(root.right) && isRed(root.right.right)) {
                root = rotateLeft(root);
            }
        }else {
            //key == root.key
            root.value = value; //避免重复插入
        }
        ...
    }
{% endhighlight %}  
~~~~~~~~
转换的过程中，我们引入了旋转的方法。
![rotateLeft]({{"/img/rotateLeft.png"}})  
![rotateRight]({{"/img/rotateRight.png"}})  
~~~~~~~~
{ % highlight Java % }  
    Node rotateLeft(Node cur) {
        Node next = cur.right;
        cur.right = next.left;
        next.left = cur;
        next.color = next.left.color; //保留原结点颜色
        next.left.color = RED;        //下降结点变红色
        return next;
    }

    Node rotateRight(Node cur) {
        Node next = cur.lext;
        cur.left = next.right;
        next.right = cur;
        next.color = next.right.color;
        next.right.color = RED;
        return next;
    }
{% endhighlight %}  
~~~~~~~~

是不是这样插入就完成了呢？
细心的肯定也发现了，如果我们插入发生在4-node的话。  
![insert into 4-node]({{"/img/insert-red4node.png"}})  
我们很容易看出来，旋转之后的结果还是没有解决。  
![rotate 4node]({{"/img/rotate-red4node.png"}})  
现在我们回想一下，上篇讨论2-3 tree和2-3-4 tree的时候，我们对插入4-node的方法是采用向上合并方法。那么，我们这里能不能用类似的方法呢？
当然是没有问题的。
![flip Colors]({{"/img/flipcolors.png"}})  
我们引入这个方法称为Flip Colors。  
~~~~~~~~
{% highlight Java %}  
    void flipColors(Node n) {
        n.color = !n.color;
        n.left.color = !n.left.color;
        n.right.color = !n.right.color;
    }
{% endhighlight %}  
~~~~~~~~
结合我们讨论2-3-4 tree时所采用的Top-down appraach，这样我们就能完善插入操作： 
~~~~~~~~ 
{% highlight Java %}  
    Node insert(Node root, Key key, Value value, boolean lr_child) {
        if(root == T.nil) {
            return new Node(key, value, RED); //总是红结点
        }
        
        if(isRed(root.left) && isRed(root.right)) {
            flipColors(root);                 //类比2-3-4 tree拆分4-node的方法
        }
        int cmp = key.compareTo(root.key);
        if(cmp < 0) {
            //key < root.key
            root.left = insert(root.left, key, value, true);
            if(isRed(root) && isRed(root.left) && !lr_child) {
                    //上图right-left情况
                    //Rotate Right操作
                    root = rotateRight(root); 
            }
            if(isRed(root.left) && isRed(root.left.left)) {
                root = rotateRight(root);
            }
        }else if(cmp > 0) {
            //key > root.key
            root.right = insert(root.right, key, value, false);
            if(isRed(root) && isRed(root.Right) && lr_child) {
                    //上图left-right情况
                    //Rotate Left操作
                    root = rotateLeft(root); 
            }
            if(isRed(root.right) && isRed(root.right.right)) {
                root = rotateLeft(root);
            }
        }else {
            //key == root.key
            root.value = value; //避免重复插入
        }
        return root;
    }
{% endhighlight %}  
~~~~~~~~
有人问，如果把Flip Colors放到最后，会怎样？
那么，此时RBT就相当于2-3 tree的实现了，因为递归回退过程中，相当于2-3 tree的拆分4-node再向上合并操作，最终没有4-node存在。 
~~~~~~~~ 
{% highlight Java %}  
    Node insert(Node root, Key key, Value value, boolean lr_child) {
        ...
        //考虑插入后结点与父结点的关系，简化。
        if(cmp < 0) ...
        else if(cmp > 0) ...
        else //cmp == 0 ...
        if(isRed(root.left) && isRed(root.right)) {
                flipColors(root); //移动到这里
        }  
        return root;
    }
{% endhighlight %}  
~~~~~~~~
有人可能会有疑问，那么根必黑特性不是被破坏了么？  
~~~~~~~~
{% highlight Java %}  
    Node insert(Node root, Key key, Value value, boolean lr_child) {
        ...
    }
    Node insert(Node root, Key key, Value value) {
        insert(root, key, value, true); //根结点lr_child没有影响
        T.root = BLACK;
    }
{% endhighlight %} 
~~~~~~~~
插入的情况还是有点多，这样删除通常也比较复杂，那么还能不能再简单一些？
那如果我们限制3-node的红结点只有一边呢？假如只有左边是红结点。

***
*因为未知原因，Red-Black Tree究极进化了，成为Left-Leaning Red-Black Tree。*  
*简称，LLRBT。*  

***

**Left-Leaning Red-Black Tree**，因为未知原因，割掉右臂的Red-Black Tree。  
我们观察结点情况：  
![right to left]({{"/img/right2left.png"}})  
那么插入操作将简化成： 
~~~~~~~~ 
{% highlight Java %}  
    Node insert(Node root, Key key, Value value) {
        if(root == T.nil) {
            return new Node(key, value, RED); //总是红结点
        }
        int cmp = key.compareTo(root.key);
        if(cmp < 0) {
            //key < root.key
            root.left = insert(root.left, key, value);
            }
        }else if(cmp > 0) {
            //key > root.key
            root.right = insert(root.right, key, value);
        }else {
            //key == root.key
            root.value = value; //避免重复插入
        }

        if(isRed(root.right)) {
            //右斜变左斜
            root = rotateLeft(root);
        }
        if(isRed(root.left) && isRed(root.left.left)) {
            //两个连续左斜的红结点,变成4-node
            root = rotateRight(root);
        }
        if(isRed(root.left) && isRed(root.right)) {
            flipColors(root);                 //类比2-3-4 tree拆分4-node的方法
        }
        return root;
    }
{% endhighlight %}  
~~~~~~~~
现在看起来是不是更加地清晰呢！  

接下来分析删除。  
我们之前分析过，删除一棵2-3-4 tree时，在底部的结点，而且是3-node，4-node结点是最容易的删除方式，直接删除就可以了。  
而类比到RBT，或者LLRBT。  
![3node or 4node]({{"/img/3-4node.png"}})  
如果删除在底部的红结点，对RBT的特性不会产生影响，删除操作可以结束。  
但是，假如底部删除的是一个2-node，或者删除的结点不在底部，情况就变得复杂。  
如果删除结点不在底部的话，我们通过前文讨论过的方法，通过寻找直接后继(IS)来替代删除结点，使删除发生在底部。  
~~~~~~~~
{% highlight Java %}  
    successor = findMin(n.right);
    n.key = successor.key;
    n.value = successor.value;
    n.right = deleteMin(n.right);
{% endhighlight %}  
~~~~~~~~
对于deleteMin操作  
~~~~~~~~
{% highlight Java %}  
    Node deleteMin(Node n) {
        if(n.left == T.nil) {
            return T.nil;
        }

        if(!isRed(n.left) && !isRed(n.left.left)) {
            //这意味着孩子是2-node，需要借点。
            //引入操作
            n = moveRedLeft(n);
        }

        n.left = deleteMin(n.left);

        fixUp(n);
    }

    fixUp(Node n) {
        //为了方便使用，整合为一个函数
        if(isRed(root.right)) {
            //右斜变左斜
            root = rotateLeft(root);
        }
        if(isRed(root.left) && isRed(root.left.left)) {
            //两个连续左斜的红结点,变成4-node
            root = rotateRight(root);
        }
        if(isRed(root.left) && isRed(root.right)) {
            flipColors(root);                 //类比2-3-4 tree拆分4-node的方法
        }
    }
{% endhighlight %}
~~~~~~~~
~~~~~~~~
{% highlight Java %}  
    Node moveRedLeft(Node n) {
        //假设父亲借点
        flipColors(n);
        if(isRed(n.right.left)) {
            //如果兄弟结点是3-node，
            //因为LLRBT，左孩子必为红
            //那不向父亲借，向兄弟借
            n.right = rotateRight(n.right);
            n = rotateLeft(n);
            flipColors(n);
        }
        return n;
    }

    Node moveRedRight(Node n) {
        flipColors(n);
        if(isRed(n.left.left)) {
            n.left = rotateRight(n);
            flipColors(n);
        }
        return n;
    }
{% endhighlight %}  
~~~~~~~~
那么现在我们可以完善删除操作：  
~~~~~~~~
{% highlight Java %}  
    delete(Node cur, Key key) {
        int cmp = key.compareTo(cur.key);
        if(cmp < 0) {
            //删除结点在右边
            if(!isRed(cur.left) && !isRed(cur.left.left)) {
                cur = moveRedLeft(cur);
            }
            cur.left = delete(cur.left, key);
        }else {
            if(isRed(cur.left)) {
                cur = leanRight(cur);
            }
            if(cmp == 0 && (cur.right == T.nil)) {
                return null;
            }
            if(!isRed(cur.right) && !isRed(cur.right.left)) {
                cur = moveRedRight(cur);
            }
            if(cmp == 0) {
                successor = findMin(cur.right);
                cur.key = successor.key;
                cur.value = successor.value;
                cur.right = deleteMin(cur.right);
            }else {
                cur.right = delete(cur.right, key);
            }
        }
        return fixUp(cur);
    }
{% endhighlight %}  
~~~~~~~~
***
#References  
  
  **[1]**.  Cormen, Thomas; Leiserson, Charles; Rivest, Ronald; Stein, Clifford (2009). "13". Introduction to Algorithms (3rd ed.). MIT Press. pp. 308–338. ISBN 978-0-262-03384-8.  
  **[2]**. [http://www.cs.princeton.edu/~rs/talks/LLRB/RedBlack.pdf](http://www.cs.princeton.edu/~rs/talks/LLRB/RedBlack.pdf)  