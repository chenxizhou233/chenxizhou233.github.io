---
title: Morris遍历
date: 2021-08-07 16:48:54
tags: 图论  
categories: C++
---

# Morris遍历， $O(1)$ 空间遍历二叉树  

在我们的学习过程中，二叉树的前、中、后序遍历相信大家并不陌生了：dfs一下就行了。  
但，在最坏情况下,整颗二叉数都会被存入栈空间中。  
为了解决这种问题，我们便引入了一种新的遍历方式：Morris遍历  
> 注意，这只是一种实现前、中、后序遍历的方式，并不是一种新的遍历规则   

<!--more-->

## 原理讲解：  
在整个过程中，我们需要维护**当前节点的位置cur**与cur的**左子树中最右端的节点pre**，并利用一些节点的右儿子的**空余的指针**，使得我们能够建立一种机制，对于没有左子树的节点只到达一次，对于有左子树的节点会到达两次来保证 $O(1)$ 的空间复杂度的。下面我们通过伪代码来讲解这一过程  
```cpp
cur = root;//cur初始化为根结点
while(cur!=null)
{
    if(cur.left==null) cur=cur.right;//如果左子树为空，将当前节点更新为右子树根结点
    else
    {
        pre = getPre(cur.left);//找到cur左子树中最右边的节点  
        if(pre.right==null)
            pre.right = cur;
            //注意，这里是Morris遍历的核心部分，它利用了空余的资源，将“存放根结点”的任务无需额外空间便完成了
        else if(pre.right==cur)
            pre.right = null;
            //还原树结构（回溯）
    }
}
``` 

我们可以发现，这个算法的核心就是利用**空闲的叶子节点的右指针**来减少空间复杂度的   
下面我们通过一张图来解释一下这个过程：
![简单的二叉树](20200216151818405.png)
- 将根节点1设置为cur。
- 因为cur（节点1）不为空，且cur（节点1）的左孩子节点2不为空，所以我们找到以节点2为根节点的左子树中最右端的节点5。
- 节点5右孩子为空，此时我们输出cur（节点1）的值，然后将节点5右孩子指向为cur，即节点1。更新cur节点为cur左孩子，即节点2。
- 因为cur（节点2）左孩子不为空，找到其左子树最右端节点4
- 节点4右孩子为空，先输出cur（节点2）的值，再将节点4右孩子指向cur（节点2）,并更新cur（节点2）为其左孩子节点4。
- 这个时候cur（节点4）的左孩子为空，所以访问其右孩子，发现右孩子指向了节点2，所以我们将cur更新为节点2。
- 这个时候我们发现cur又指向了节点2，所以左孩子节点4不为空，我们再次找到左子树中最右端节点4，但是这个时候节点4的右孩子指向了cur，所以我们将其删除，即节点4右孩子指向为空，恢复原来的树结构。并且由于已经访问了左孩子和根节点，所以这个时候我们访问其右孩子节点5。
- ……  
## 代码实现
### 前序
```cpp
void MorrisPreOrderTraverse(Node *root)
{
	if (root == NULL)
	{
		return;
	}
 
	Node *cur = root;
	Node *pre = NULL;
 
	while (cur != NULL)
	{
		pre = cur->left;
		if (pre != NULL)
		{
			while(pre->right != NULL && pre->right != cur)
			{
				pre = pre->right;
			}
			if (pre->right == NULL)
			{
				pre->right = cur;		// 空闲指针
				cout<<cur->val<<" ";	// 打印结点值的顺序稍微调整
				cur = cur->left;
				continue;
			}
			else
			{
				pre->right = NULL;
			}
		}
		else
		{
			cout<<cur->val<<" ";
		}
		cur = cur->right;
	}
}
```
### 中序  
```cpp
void MorrisInOrderTraverse(Node *root)
{
	if (root == NULL)
	{
		return;
	}
 
	Node *cur = root;
	Node *pre = NULL;
 
	while (cur != NULL)
	{
		pre = cur->left;
		if (pre != NULL)
		{
			while(pre->right != NULL && pre->right != cur)
			{
				pre = pre->right;
			}
			if (pre->right == NULL)
			{
				pre->right = cur;		// 空闲指针
				cur = cur->left;
				continue;
			}
			else
			{
				pre->right = NULL;
			}
		}
		cout<<cur->val<<" ";
		cur = cur->right;
	}
}
```
### 后序
```cpp
Node* reverseEdge(Node *root)
{
	Node *pre = NULL;
	Node *next = NULL;
 
	while(root != NULL)
	{
		next = root->right;
		root->right = pre;
		pre = root;
		root = next;
	}
 
	return pre;
}
 
// 逆序打印左子树右边界
void printEdge(Node *root)
{
	Node *lastNode = reverseEdge(root);
	Node *cur = lastNode;
 
	while (cur != NULL)
	{
		cout<<cur->val<<" ";
		cur = cur->right;
	}
	reverseEdge(lastNode);
}
 
// Morris后序遍历 (左 -> 右 -> 根)
void MorrisPostOrderTraverse(Node *root)
{
	if (root == NULL)
	{
		return;
	}
 
	Node *cur = root;
	Node *pre = NULL;
 
	while (cur != NULL)
	{
		pre = cur->left;
		if (pre != NULL)
		{
			while(pre->right != NULL && pre->right != cur)
			{
				pre = pre->right;
			}
			if (pre->right == NULL)
			{
				pre->right = cur;		// 空闲指针
				cur = cur->left;
				continue;
			}
			else
			{
				pre->right = NULL;
				printEdge(cur->left);
			}
		}
		cur = cur->right;
	}
	printEdge(root);
}
```

## 参考资料  
>感谢以下博客，为本文提供了资料  
>1.[遍历的c++实现](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/solution/xiang-jie-cshi-xian-morriszhong-xu-bian-li-jie-fa-/)   
>2. [图源侵删](https://blog.csdn.net/danmo_wuhen/article/details/104339630)  

> 感谢以下博客，帮助我更好的理解。
>1.[神级遍历——morris](https://zhuanlan.zhihu.com/p/101321696)
>2.[Morris遍历](https://blog.csdn.net/weixin_42638946/article/details/118788173)
