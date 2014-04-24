---
date: "2014-04-23"
layout: post
title: "二叉树非递归遍历简洁版"
categories: 算法
tags: 算法
---

本文转载自[(http://jianshu.io/p/49c8cfd07410](http://jianshu.io/p/49c8cfd07410)

解决二叉树的很多问题的方案都是基于对二叉树的遍历。遍历二叉树的前序，中序，后序三大方法算是计算机科班学生必写代码了。其递归遍历是人人都能信手拈来，可是在手生时写出非递归遍历恐非易事。正因为并非易事，所以网上出现无数的介绍二叉树非递归遍历方法的文章。可是大家需要的真是那些非递归遍历代码和讲述吗？代码早在学数据结构时就看懂了，理解了，可为什么我们一而再再而三地忘记非递归遍历方法，却始终记住了递归遍历方法？

三种递归遍历对遍历的描述，思路非常简洁，最重要的是三种方法完全统一，大大减轻了我们理解的负担。而我们常接触到那三种非递归遍历方法，除了都使用栈，具体实现各有差异，导致了理解的模糊。本文给出了一种统一的三大非递归遍历的实现思想。

##三种递归遍历##

	//前序遍历
	void preorder(TreeNode *root, vector<int> &path)
	{
	    if(root != NULL)
	    {
	        path.push_back(root->val);
	        preorder(root->left, path);
	        preorder(root->right, path);
	    }
	}


	//中序遍历
	void inorder(TreeNode *root, vector<int> &path)
	{
	    if(root != NULL)
	    {
	        inorder(root->left, path);
	        path.push_back(root->val);
	        inorder(root->right, path);
	    }
	}

	//后续遍历
	void postorder(TreeNode *root, vector<int> &path)
	{
	    if(root != NULL)
	    {
	        postorder(root->left, path);
	        postorder(root->right, path);
	        path.push_back(root->val);
	    }
	}
由上可见，递归的算法实现思路和代码风格非常统一,关于“递归”的理解可见[《人脑理解递归》](http://jianshu.io/p/4db970d8ddc1)。

##教科书上的非递归遍历##

	//非递归前序遍历
	void preorderTraversal(TreeNode *root, vector<int> &path)
	{
	    stack<TreeNode *> s;
	    TreeNode *p = root;
	    while(p != NULL || !s.empty())
	    {
	        while(p != NULL)
	        {
	            path.push_back(p->val);
	            s.push(p);
	            p = p->left;
	        }
	        if(!s.empty())
	        {
	            p = s.top();
	            s.pop();
	            p = p->right;
	        }
	    }
	}
	//非递归中序遍历
	void inorderTraversal(TreeNode *root, vector<int> &path)
	{
	    stack<TreeNode *> s;
	    TreeNode *p = root;
	    while(p != NULL || !s.empty())
	    {
	        while(p != NULL)
	        {
	            s.push(p);
	            p = p->left;
	        }
	        if(!s.empty())
	        {
	            p = s.top();
	            path.push_back(p->val);
	            s.pop();
	            p = p->right;
	        }
	    }
	}
	//非递归后序遍历-迭代
	void postorderTraversal(TreeNode *root, vector<int> &path)
	{
	    stack<TempNode *> s;
	    TreeNode *p = root;
	    TempNode *temp;
	    while(p != NULL || !s.empty())
	    {
	        while(p != NULL) //沿左子树一直往下搜索，直至出现没有左子树的结点
	        {
	            TreeNode *tempNode = new TreeNode;
	            tempNode->btnode = p;
	            tempNode->isFirst = true;
	            s.push(tempNode);
	            p = p->left;
	        }
	        if(!s.empty())
	        {
	            temp = s.top();
	            s.pop();
	            if(temp->isFirst == true)   //表示是第一次出现在栈顶
	            {
	                temp->isFirst = false;
	                s.push(temp);
	                p = temp->btnode->right;
	            }
	            else  //第二次出现在栈顶
	            {
	                path.push_back(temp->btnode->val);
	                p = NULL;
	            }
	        }
	    }
	}

看了上面教科书的三种非递归遍历方法，不难发现，后序遍历的实现的复杂程度明显高于前序遍历和中序遍历，前序遍历和中序遍历看似实现风格一样，但是实际上前者是在指针迭代时访问结点值，后者是在栈顶访问结点值，实现思路也是有本质区别的。而这三种方法最大的缺点就是都使用嵌套循环，大大增加了理解的复杂度。

##更简单的非递归遍历二叉树的方法##

	//更简单的非递归前序遍历
	void preorderTraversalNew(TreeNode *root, vector<int> &path)
	{
	    stack< pair<TreeNode *, bool> > s;
	    s.push(make_pair(root, false));
	    bool visited;
	    while(!s.empty())
	    {
	        root = s.top().first;
	        visited = s.top().second;
	        s.pop();
	        if(root == NULL)
	            continue;
	        if(visited)
	        {
	            path.push_back(root->val);
	        }
	        else
	        {
	            s.push(make_pair(root->right, false));
	            s.push(make_pair(root->left, false));
	            s.push(make_pair(root, true));
	        }
	    }
	}
	//更简单的非递归中序遍历
	void inorderTraversalNew(TreeNode *root, vector<int> &path)
	{
	    stack< pair<TreeNode *, bool> > s;
	    s.push(make_pair(root, false));
	    bool visited;
	    while(!s.empty())
	    {
	        root = s.top().first;
	        visited = s.top().second;
	        s.pop();
	        if(root == NULL)
	            continue;
	        if(visited)
	        {
	            path.push_back(root->val);
	        }
	        else
	        {
	            s.push(make_pair(root->right, false));
	            s.push(make_pair(root, true));
	            s.push(make_pair(root->left, false));
	        }
	    }
	}
	//更简单的非递归后序遍历
	void postorderTraversalNew(TreeNode *root, vector<int> &path)
	{
	    stack< pair<TreeNode *, bool> > s;
	    s.push(make_pair(root, false));
	    bool visited;
	    while(!s.empty())
	    {
	        root = s.top().first;
	        visited = s.top().second;
	        s.pop();
	        if(root == NULL)
	            continue;
	        if(visited)
	        {
	            path.push_back(root->val);
	        }
	        else
	        {
	            s.push(make_pair(root, true));
	            s.push(make_pair(root->right, false));
	            s.push(make_pair(root->left, false));
	        }
	    }
	}

以上三种遍历实现代码行数一模一样，如同递归遍历一样，只有三行核心代码的先后顺序有区别。为什么能产生这样的效果？下面我将会介绍。

##有重合元素的局部有序一定能导致整体有序##

这就是我得以统一三种更简单的非递归遍历方法的基本思想：**有重合元素的局部有序一定能导致整体有序**。
如下这段序列，局部`2 3 4`和局部`1 2 3`都是有序的，但是不能由此保证整体有序。
![1.jpg](https://www.dropbox.com/s/bq15fbht70afjbi/1.png)