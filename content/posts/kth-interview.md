---
title: "K-th Problems in Interview"
date: 2018-03-01
slug: "kth-interview"
tags:
  - "Algorithm"
---

K-th问题是说在给定数据集中，找到排名第k个的数据。通常这个数据集极大是没办法存在内存中，或者遍历一遍数据集的时间代价很高。通常解决这种方法是维护一个k大小的最大堆，这样我们可以一直维护top k个数据。这种方法比较常见且简单，这里我想说得是一般这种K-th类问题是可以通过Binary Search解决。

## Binary Search

二分查找，是非常简单的算法。但是这种算法应用非常灵活，首先想一次性写对二分查找并不是一件容易的事情。这里我想简单总结一下二分查找的使用背景。众所周知，最最常见应用二分查找的地方就是对于一个已经排好序的数组中，找到一个我们已知的值的位置。这个问题背后隐藏着什么呢？记得我在初次学习二分查找的时候，看到有人总结说二分查找的应用条件就是这个问题是否可以证明是“单调的”。对于数组查找中，我们可以简化问题为一个函数`f`，`f(index)`就是对于查找`index`的值是否大于等于目标值，这样我们就发现函数`f`在定义域`[0, n)`是满足单调性质的，因为必然有个界限让小于的`f`为false，大于等于的时候是true（当然这个界限也有可能不存在，就是我们没有找到这个值的时候）。再举个例子，我们想找到实数取决一个满足`f(x)=y`的值，也可以对整个实数区域进行二分，然后求解`x`，就像解方程一样。

说完了背景，再说说写法。首先是整数的写法：

因为整数的二分会自动去掉小数部分，这就会遇到一个头疼的问题，如果当`low = k, high = k + 1` 这时候我们做二分，中间值永远是`k`。如果我们写的代码是`while (low < high) {if (f(mid)) { low = mid; } else { high = mid - 1; }}` 且这个时候`f(mid)`为true，那么我们永远也无法跳出这个循环。解决这个问题有两种思路：1. 因为我们停止的条件是`low == high`所以不妨把循环条件设置为 `while (low + 1 < high)` 但是这又会引起一个问题，我们需要在最后再double check一下low和high我们需要的答案是什么。 2. 也可以把二分的结果向上取整 `mid = (low + high + 1) / 2` 这样在遇到这种情况是我们总会选择`high`。所以在写完二分解法后最好想一下，两个答案的时候程序能不能成功运行出结果。第二个是值域分布的问题，有可能我们的判断函数`f(x)`永远都是false，这样的结果是我们最后的二分结果也是错的。

下面是实数二分的写法：

这里就不会遇到那个头疼问题，我们只要保证精度就可以了 `while (left < high && Math.abs(left - high) < 1e-5)` 但是我不喜欢这么写，会因为精度的问题陷入死循环。我们单纯循环100次就可以把精度控制在小数点后30位。

## K-th with Binary Search

这里说一下为什么K-th问题都可以借用二分查找解决。因为K-th问题必然是单调性的，因为我们有函数`f(x)=k`表示`x`在数据集中的排序为`k`，那么比x大的数字，`f(x)`就会变大，反之亦然，单调性很容易证明。所以我们只要在合理的时间计算出`x`的排序，就可以二分查找出排序为`k`的数值。下面我们来看几道题：

[Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/description/) 给了一个数组，让找出两两pair中相差值为k-th小的值。我们可以对数组排序，然后二分这个相差值。然后根据这个值找出有多少个pair比这个值小。这里注意并不是如果比这个值小的元素有`k-1`个就证明了就是答案，因为我们不知道这个相差值是否真的存在。所以我们要找到答案为`k-1`且这个值最小的那个。因为这个值是处于一个分界点，比这个值小排序的序号就减少1，就说明了这个差值是存在的。这里我们用sliding window来找到多少个差值比target要小。

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 ``` | ``` public int smallestDistancePair(int[] nums, int k) { 	Arrays.sort(nums); 	int low = 0, high = Integer.MAX_VALUE; 	while (low < high) { 		int mid = (low + high) / 2; 		int begin = 0, end = 0, cnt = 0; 		for (; end < nums.length; ++ end) { 			while (begin < nums.length && nums[end] - nums[begin] > mid) { 				begin++; 			} 			cnt += (end - begin); 		} 		if (cnt >= k) { 			high = mid; 		} else { 			low = mid + 1; 		} 	} 	return low; } ``` |

[K-th Smallest Prime Fraction](https://leetcode.com/problems/k-th-smallest-prime-fraction/description/) 给一个排序好的素数数组，对于 `p < q find p / q` 找出所有值中第k小的。同样，这里我们对实数值域进行二分，找到多少个分数比当前值小，同样找到这些满足条件中的最小值。我们也可以通过比较巧妙的线性方法的找出所有满足小于查找值的分数个数。

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 ``` | ``` public int[] kthSmallestPrimeFraction(int[] A, int K) { 	int n = A.length; 	double low = (double)A[0]/A[n - 1]; 	double high = (double)A[n - 1]/A[0]; 	Set<Integer> set = new HashSet<>(); 	for (int a : A) set.add(a); 	for (int itr = 0; itr < 100; ++itr) { 		double mid = (low + high) * 0.5; 		int i = 0, j = 0, cnt = 0; 		for (; i < n; ++ i) { 			for (; j < n; ++ j) { 				if (i < j && A[i] < mid * A[j]) { 					cnt += (n - j); 					break; 				} 			} 		} 		if (cnt < K) { 			low = mid; 		} else { 			high = mid; 		} 	} 	for (int a : A) { 		double p = Math.round(low * (double)a); 		if (Math.abs(p - low * (double)a) < 1e-5 && set.contains((int)p)) { 			return new int[]{(int)p, a}; 		} 	} 	return null; } ``` |

[Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/description/) 给一个行和列都排序的矩阵，找到其中第k小的。我们二分来查找这个值，然后通过每行进行二分查找来统计多少个值比这个数要小。

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 ``` | ``` public int kthSmallest(int[][] matrix, int k) { 	int n = matrix.length; 	int low = matrix[0][0], high = matrix[n - 1][n - 1]; 	while (low < high) { 		int mid = (low + high) / 2; 		int count = 0; 		for (int i = 0; i < n; ++ i) { 			int l = 0, h = n - 1; 			while (l < h) { 				int m = (l + h + 1) / 2; 				if (matrix[i][m] <= mid) { 					l = m; 				} else { 					h = m - 1; 				} 			} 			if (matrix[i][l] <= mid) { 				count += l + 1; 			} 		} 		if (count < k) { 			low = mid + 1; 		} else { 			high = mid; 		} 	} 	return low; } ``` |

[Kth Smallest Number in Multiplication Table](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/description/) 和上道题类似，每行换成了等比数列。免去了进行二分查找统计，直接算除法就可以统计数量。

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 ``` | ``` public int findKthNumber(int m, int n, int k) { 	int low = 1, high = m * n; 	while (low < high) { 		int mid = (low + high) >>> 1; 		int count = 0; 		for (int i = 1; i <= m; ++ i) { 			count += Math.min(n, mid / i); 		} 		if (count < k) { 			low = mid + 1; 		} else { 			high = mid; 		} 	} 	return low; } ``` |
