---
title: "Binary Search Tree in Interview"
date: 2018-02-25
slug: "bst"
tags:
  - "Algorithm"
---

Binary Search Tree (简称BST) 是最简单的搜索树，很多高级树用法都是以它为基础。因为设计它的算法很多都很巧妙，记得在面试的时候突然遇到一道BST的题卡住了。由于BST的结构特殊，所以有关它的题目都很灵活，想借此机会总结一下。

## 递归与递推

众所周知，BST是天生具有递归特性的，因为的BST的任一子树都是BST。所以很多题目都是利用这一性质。比如将BST有序打印出来就可以使用inorder遍历：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 ``` | ``` public void recursive(TreeNode root) {     if (root == null) return;     recursive(root.left);         /*         Do something...         */     recursive(root.right); } ``` |

当然也可以通过栈来改写为递推，这里是将未处理的结点存入栈中。这里需要介绍一种`pushAll`操作。就是将该结点的所有左结点全部压入栈。然后每次取出结点后对右结点进行`pushAll`操作。[Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 ``` | ``` public List<Integer> inorderTraversal(TreeNode root) { 	List<Integer> ans = new ArrayList<>(); 	Stack<TreeNode> stack = new Stack<>(); 	while(root != null) { 		stack.push(root); 		root = root.left; 	} 	while (!stack.isEmpty()) { 		TreeNode cur = stack.pop(); 		ans.add(cur.val); 		cur = cur.right; 		while (cur != null) { 			stack.push(cur); 			cur = cur.left; 		} 	} 	return ans; } ``` |

了解到这个特性后，我们就可以将BST改写成`Iterator` - `next()`取出下一个最小值，`hasNext()`是否存在结点。[Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/description/)：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 ``` | ``` public class BSTIterator {     private Deque<TreeNode> stack;      public BSTIterator(TreeNode root) {         stack = new ArrayDeque<>();         pushAll(root);     }      /** @return whether we have a next smallest number */     public boolean hasNext() {         return !stack.isEmpty();     }      /** @return the next smallest number */     public int next() {         TreeNode cur = stack.pop();         pushAll(cur.right);         return cur.val;     }      private void pushAll(TreeNode root) {         while (root != null) {             stack.push(root);             root = root.left;         }     } } ``` |

在Java的`Iterator`是也可以支持`remove()`操作 - 删除上一个访问的结点。BST删除结点的操作也可以用递归算法来解决（当然也可用平衡二叉树删除结点进行分类的解法，但是递归的解法更加巧妙）。首先我们找到被删除的结点，然后把它替换成其右子树的最小值`m`（目的是为了保证删除后的右子树满足比当前结点大）。然后我们再递归删除右子树的`m`结点。[Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/description/)：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 ``` | ``` public TreeNode deleteNode(TreeNode root, int key) { 	if (root == null) { 		return null; 	} 	if (key < root.val) { 		root.left = deleteNode(root.left, key); 	}  else if (key > root.val) { 		root.right = deleteNode(root.right, key); 	} else { 		if (root.left == null) { 			return root.right; 		} else if (root.right == null) { 			return root.left; 		} else { 			TreeNode min = findMin(root.right); 			// copy 			root.val = min.val; 			root.right = deleteNode(root.right, min.val); 		} 	} 	return root; }  private TreeNode findMin(TreeNode root) { 	TreeNode cur = root; 	while (cur.left != null) { 		cur = cur.left; 	} 	return cur; } ``` |

## 查找

上文中我们知道BST是可以有序输出的，所以很多针对有序数据的算法都可以套在BST上。比如，二分查找，第k大问题，等等。

如果查找的结点存在于BST上，我们很容易可以将它找出。但是如果需要查找的结点不存在与树上，我们希望返回最接近的结点，这种问题该如何解决呢？首先我们要想一个问题，对于一个在BST上的结点，比它大的值可能存在于哪里？应该是**存在于这个结点的右子树或父结点（如果存在）的右子树**。但是这么思考的问题是我们需要跟踪父结点的情况，这个如果不借助其他数据结构存储很难实现。我们不如换一种思路：当访问到结点时，如果值大于target就找保存左子树的最接近target的值，然后取这两个值最接近target的一个；反之就从右子树来找。同样借助递归的特性我们可以解决这个问题。[Closest Binary Search Tree Value](https://leetcode.com/problems/closest-binary-search-tree-value/description/)

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 ``` | ``` public int closestValue(TreeNode root, double target) { 	if (root == null) { 		return -1; 	} 	if (Math.abs((double)root.val - target) < 1e-6) { 		return root.val; 	} 	if ((double)root.val < target && root.right != null) { 		int r = closestValue(root.right, target); 		if (Math.abs(target - (double)r) < Math.abs(target - (double)root.val)) { 			return r; 		} 		return root.val; 	} else if ((double)root.val > target && root.left != null) { 		int r = closestValue(root.left, target); 		if (Math.abs(target - (double)r) < Math.abs(target - (double)root.val)) { 			return r; 		} 		return root.val; 	} 	return root.val; } ``` |

借用这种想法，我们还可以解决类似的问题。比如将BST分成两个BST，一个所有的值小于等于target，另一个大于target。对于当前结点如果小于等于target，其左边子树都是小于等于target，所以将右子树分成的结果的第一部分连接到当前结点的右边；如果大于target，说明右子树都是大于target，需要将左子树的结果的第二部分连入左边。[Split BST](https://leetcode.com/problems/split-bst/description/)：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 ``` | ``` public TreeNode[] splitBST(TreeNode root, int V) { 	if (root == null) { 		return new TreeNode[]{null, null}; 	} 	TreeNode[] res; 	if (root.val <= V) { 		res = splitBST(root.right, V); 		root.right = res[0]; 		res[0] = root; 	} else { 		res = splitBST(root.left, V); 		root.left = res[1]; 		res[1] = root; 	} 	return res; } ``` |

对于第k大的问题，可以通过inorder遍历来解决。[Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/description/)：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 ``` | ``` int count = 0; public int kthSmallest(TreeNode root, int k) { 	return solve(root, k); }  private int solve(TreeNode root, int k) { 	if (root == null) { 		return -1; 	} 	int r = solve(root.left, k); 	if (r != -1) return r; 	count++; 	if (count == k) { 		return root.val; 	} 	r = solve(root.right, k); 	return r; } ``` |

另一种变形是，在BST找到接近target的k个数。这个可以通过inorder来找到，但是这里介绍一种高效的方法借助额外的数据结构来解决。通过栈来存储比target小的结点，另一个栈存储大于等于target的结点。然后用类似two pointer的方法找到接近target的k个数。[Closest Binary Search Tree Value II](https://leetcode.com/problems/closest-binary-search-tree-value-ii/description/)：

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 ``` | ``` public List<Integer> closestKValues(TreeNode root, double target, int k) { 	Deque<TreeNode> greater = new ArrayDeque<>(), less = new ArrayDeque<>(); 	TreeNode cur = root; 	while (cur != null) { 		if (cur.val < target) { 			less.push(cur); 			cur = cur.right; 		} else { 			greater.push(cur); 			cur = cur.left; 		} 	} 	List<Integer> list = new ArrayList<>(); 	while (k > 0) { 		if (greater.isEmpty() && less.isEmpty()) { 			break; 		} else if (greater.isEmpty()) { 			list.add(lessNext(less)); 		} else if (less.isEmpty()) { 			list.add(greaterNext(greater)); 		} else if (Math.abs(less.peek().val - target) < Math.abs(greater.peek().val - target)) { 			list.add(lessNext(less)); 		} else { 			list.add(greaterNext(greater)); 		} 		k--; 	} 	return list; }  private int lessNext(Deque<TreeNode> stack) { 	TreeNode top = stack.pop(); 	int r = top.val; 	top = top.left; 	while (top != null) { 		stack.push(top); 		top = top.right; 	} 	return r; }  private int greaterNext(Deque<TreeNode> stack) { 	TreeNode top = stack.pop(); 	int r = top.val; 	top = top.right; 	while (top != null) { 		stack.push(top); 		top = top.left; 	} 	return r; } ``` |
