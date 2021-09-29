### 1、Array 题型

~~~bash
开闭需要根据题目的具体要求来确定，不能拥有多个意义
在处理的时候非零的相对位置需不需要保持一致
~~~

**重点：**

- **二分的区间边界不清楚**（开区间、闭区间和半开半闭）
- **二分查找的结果不明确**（是while (l < r) 还是 while (l <= r) 、）

#### 双指针同向

~~~java
通用步骤：
    1、初始化两个指针 第一个为 i = 0，第二个为j = 0;
    2、while(i<=j) {
        1、如果我们需要 array[j]，array[i] = array[j]，把 i,j 向前移动一步
        2、否则直接 j 往前挪动
        3、i 就是新的 array 的值 [0,i)
    }
~~~

#### 双指针不同向

~~~
通用步骤：
    1、初始化两个指针 第一个为 i = 0，第二个为j = array.length - 1;
    2、while(i<=j) {
        1、在 array[i] 和 array[j] 的基础上决定应该做什么
        2、在他的方向上至少移动一个指针
    }
~~~

#### 1、盛水最多的容器

~~~java
public class Solution {
    public int maxArea(int[] height) {
        int l = 0, r = height.length - 1;
        int ans = 0;
        while (l < r) {
            int area = Math.min(height[l], height[r]) * (r - l);
            ans = Math.max(ans, area);
            if (height[l] <= height[r]) {
                ++l;
            }
            else {
                --r;
            }
        }
        return ans;
    }
}
~~~

#### 2、接雨水

~~~java
class Solution {
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int ans = 0;
    int left_max = 0, right_max = 0;
    while (left < right) {
        if (height[left] < height[right]) {
            if (height[left] >= left_max) {
                left_max = height[left];
            } else {
                ans += (left_max - height[left]);
            }
            ++left;
        } else {
            if (height[right] >= right_max) {
                right_max = height[right];
            } else {
                ans += (right_max - height[right]);
            }
            --right;
        }
    }
    return ans;
}
}
~~~

#### 3、移动0

~~~java
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length, left = 0, right = 0;
        while (right < n) {
            if (nums[right] != 0) {
                swap(nums, left, right);
                left++;
            }
            right++;
        }
    }

    public void swap(int[] nums, int left, int right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;
    }
}
~~~

#### 4、删除排序数组中的重复项 II

~~~java
class Solution {
    
    public int removeDuplicates(int[] nums) {
        int j = 1, count = 1;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] == nums[i - 1]) {
                count++;
            } else {
                count = 1;
            }
            if (count <= 2) {
                nums[j++] = nums[i];
            }
        }
        return j;
    }
}
~~~

5、删除字符串中的所有相邻重复项

~~~java
class Solution {
    public String removeDuplicates(String S) {
        StringBuffer stack = new StringBuffer();
        int top = -1;
        for (int i = 0; i < S.length(); ++i) {
            char ch = S.charAt(i);
            if (top >= 0 && stack.charAt(top) == ch) {
                stack.deleteCharAt(top);
                --top;
            } else {
                stack.append(ch);
                ++top;
            }
        }
        return stack.toString();
    }
}
~~~

### 2、二分查找

> 针对有效区间内的 O(logN) 搜索方式，常见于已经排好序的 Array

#### 1、找一个准确值

- 循环条件：l <= r
- 缩减空间搜索条件：l = mid + 1,r = mid - 1

~~~java
public int binarySearch(int[] arr,int k) {
    int l = 0;
    int r = arr.length - 1;
    while (l <= r) {
        int mid = l + (r - l) / 2;
        if (arr[mid] == k) {
            return mid;
        } else if (arr[mid] > k) {
            r = mid - 1;
        } else {
            l = mid + 1;
        }
    }
    return -1;
}
~~~

#### 2、找一个模糊值

需要满足的条件：

1. 每次都要缩减搜索区域
2. 每次缩减不能排除潜在答案

- mid = l + (r - l + 1) / 2
- 循环条件：l < r   【因为如果 l == r 那么则存在某种情况还是在原地】
- 缩减搜索空间：l = mid,r = mid - 1 【从左到右依次排序、不能排除潜在答案，所以使用后边这种算法、一般用于找某个东西最后一次出现的位置】或者 l = mid + 1,r = mid【从左到右依次排序、不能排除潜在答案，所以使用后边这种算法、一般用于找某个东西第一次出现的位置】

~~~java
public int binarySearch(int[] arr,int k) {
    int l = 0;
    int r = arr.length - 1;
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (arr[mid] < k) {
            l = mid + 1;
        } else (arr[mid] > k) {
            r = mid;
        }
    }
    return l;
}
~~~

#### 3、找最接近某个值的数

需要满足的条件：

1. 每次都要缩减搜索区域
2. 每次缩减不能排除潜在答案

- 循环条件：l < r - 1 
- 缩减搜索空间：l = mid,r = mid

~~~java
public int binarySearch(int[] arr,int k) {
    int l = 0;
    int r = arr.length - 1;
    while (l < r - 1) {
        int mid = l + (r - l) / 2;
        if (arr[mid] < k) {
            l = mid + 1;
        } else (arr[mid] > k) {
            r = mid;
        }
    }
    if (arr[r] < k) {
       return r; 
    } else if (arr[l] > k) {
       return l;   
    } else {
        return k - arr[l] < arr[r] - k ? l : r; 
    }
}
~~~

#### 4、找最长重复子串

~~~java
// 注意子串的长度小于（整个子串只会出现一次）所以为左闭右开
   mid = l + (r - l + 1) / 2
public int longestRepeatingSubstring(String s) {
    int l = 0;
    int r = s.length - 1;
    while (l < r) {
        int mid = l + (r - l + 1) / 2;
        if (f(s,mid)) {
            l = mid ;
        } else {
            r = mid - 1;
        }
    }
    return l;
}

public boolean f(String s,int length) {
    Set<String> seen = new HashSet<>();
    for (int i = 0;i <= s.length() - length;i++) {
        int j = i  + l -1;
        String sub = s.substring(i,j+1);
        if (seen.contains(sub)) {
            return true;
        }
    }
    return false;
}
~~~

### 3、链表

> **双指针问题**
>
> **两个指针指向 LinkedList 节点，不再是 index**
>
> **两个指针同向**
>
> 1. **一个快一个慢，距离隔开多少**
> 2. **两个指针的移动速度**

#### 1、找中间节点

~~~java
public ListNode linkedListMiddleNode (ListNode head) {
    ListNode i = head,j=head;
    while (j != null && j.next != null) {
        i = i.next;
        j = j.next.next;
    }
    return i;
}
~~~

2、找到倒数第 k 个节点

~~~java
// 两个指针先隔开 k 个位置 每次相同速度往前，慢指针停在倒数第 k 个节点
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode slow = head,fast = head;
        for (int i = 0;i < k;i++) {
           fast = fast.next; 
        }
        while (fast != null) {
            i = i.next;
            j = j.next;
        }
        return i;
    }
~~~

#### 2、递归

> 1.  问子问题要答案
> 2. 在当前层做一些事情
> 3. 返回结果

~~~java
public ListNode reverse(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode reverse_head = reverse(head.next);
    head.next.next = head;
    head.next = null;
    return reverse_head;
}
~~~

### 4、Stack

> 适用于需要记录之前的状态、每个状态用O(1) 次数

- 在 arr[i] 左侧所有比 arr[i] 大的数
- 递归之前的函数状态（Call Stack）

#### 1、每日温度

~~~java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        // 定义变量、初始化结果集、初始化堆栈
        int length = T.length;
        int[] ans = new int[length];
        Deque<Integer> stack = new ArrayDeque<Integer>();
        // 遍历数组；根据条件选择继续 push 还是弹出并计算当前距离
        // 对于每一个元素 做一个操作
        for (int i = 0;i < length;i++) {
            int temp = T[i]; // 标识当前到了数组的哪个位置
            while (!stack.isEmpty() && temp > T[stack.peek()]) {
                int index = stack.pop();
                ans[index] = i -index;
            }
        }
        stack.push(i);
    }
}
~~~

#### 2、行星碰撞

~~~java
    public int[] asteroidCollision(int[] asteroids) {
        Deque<Integer> stack = new ArrayDeque<Integer>();
        for (int ans : asteroids) {
            // 利用正向来做
            if (ans > 0) {
                stack.push(ans);
            } else {
                while (!stack.isEmpty() && stack.peek() > 0 && stack.peek() < -ans) {
                    stack.pop();
                }
                if (!stack.isEmpty() && stack.peek() == -ans) {
                    stack.pop();
                } else if (stack.isEmpty() || stack.peek() < 0) {
                    stack.push(ans);
                }
            }
        }
        int[] res = new int[stack.size()];
        for (int i = res.length - 1; i >= 0; i--) {
            res[i] = stack.pop();
        }
        return res;
    }
~~~



### 5、哈希表

#### 1、和为K的子数组

~~~java
public class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0, pre = 0;
        HashMap < Integer, Integer > mp = new HashMap < > ();
        mp.put(0, 1);
        for (int i = 0; i < nums.length; i++) {
            pre += nums[i];
            if (mp.containsKey(pre - k)) {
                count += mp.get(pre - k);
            }
            mp.put(pre, mp.getOrDefault(pre, 0) + 1);
        }
        return count;
    }
}
~~~

### 6、二叉树

> 类似链表；内存中不连续的数据；由各个节点的reference连接起来

#### 1、广度优先

> 按照“层”的概念进行搜索；**适合解决与层有关的Tree的题目**

##### 1、广度优先搜索

~~~java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
            //初始化结果数组
            //需要一个既能进去又能出去的数据结构,维护每一层遍历次数
            List<List<Integer>> result = new ArrayList<>();
            if (root == null) {
                return result;
            }
            Queue<TreeNode> queue = new LinkedList();
            queue.offer(root);
            while (!queue.isEmpty()) {
                List<Integer> list = new ArrayList<>();
                int size = queue.size();
                for (int i = 0; i < size; i++) {
                    TreeNode treeNode = queue.poll();
                    list.add(treeNode.val);
                    //本层完了以后，加入下一层的节点，为下次遍历作准备
                    if (treeNode.left != null) {
                        queue.offer(treeNode.left);
                    }
                    if (treeNode.right != null) {
                        queue.offer(treeNode.right);
                    }
                }
                result.add(list);
            }
            return result;
        }
}
~~~

##### 2、二叉树最大深度

~~~java
class Solution {
    // 直接进行递归
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        } 
            int leftHeight = maxDepth(root.left);
            int rightHeight = maxDepth(root.right);
            return Math.max(leftHeight, rightHeight) + 1;
    }
    
    // 利用广度优先搜索的普遍解法
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int depth = 0;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0;i<size;i++) {
                TreeNode cuur = queue.poll();
                if (cuur.left != null) {
                    queue.offer(cuur.left);
                }
                if (cuur.right != null) {
                    queue.offer(cuur.right);
                }
            }
            depth++;
        }
        return depth;
    }
}
~~~

##### 3、二叉树最小深度

~~~java
class Solution {
    // 直接进行递归
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        if (root.left == null && root.right == null) {
            return 1;
        }

        int min_depth = Integer.MAX_VALUE;
        if (root.left != null) {
            min_depth = Math.min(minDepth(root.left), min_depth);
        }
        if (root.right != null) {
            min_depth = Math.min(minDepth(root.right), min_depth);
        }

        return min_depth + 1;
    }
    
    // 利用广度优先搜索的思想（用一个扩展的带有当前深度的节点）
    class QueueNode {
        TreeNode node;
        int depth;

        public QueueNode(TreeNode node, int depth) {
            this.node = node;
            this.depth = depth;
        }
    }

    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Queue<QueueNode> queue = new LinkedList<QueueNode>();
        queue.offer(new QueueNode(root, 1));
        while (!queue.isEmpty()) {
            QueueNode nodeDepth = queue.poll();
            TreeNode node = nodeDepth.node;
            int depth = nodeDepth.depth;
            if (node.left == null && node.right == null) {
                return depth;
            }
            if (node.left != null) {
                queue.offer(new QueueNode(node.left, depth + 1));
            }
            if (node.right != null) {
                queue.offer(new QueueNode(node.right, depth + 1));
            }
        }

        return 0;
    }
}
~~~



##### 4、二叉树的右视图

~~~java
// 不一定要拿到完美的最优解  按照结果导向转变方式
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
            List<Integer> result = new ArrayList<>();
            if (root == null) {
                return result;
            }
            Queue<TreeNode> queue = new LinkedList();
            queue.offer(root);
            while (!queue.isEmpty()) {
                int size = queue.size();
                result.add(queue.peek().val);
                for (int i = 0; i < size; i++) {
                    TreeNode treeNode = queue.poll();
                    //本层完了以后，加入下一层的节点，为下次遍历作准备
                    if (treeNode.right != null) {
                        queue.offer(treeNode.right);
                    }
                    if (treeNode.left != null) {
                        queue.offer(treeNode.left);
                    }
                }
            }
            return result;
        }
}
~~~

##### 5、对称二叉树判断

###### 5.1、递归

~~~java
class Solution {
    // 转换为两个二叉树的
    public boolean isSymmetric(TreeNode root) {
        return getResult(root,root);
    }
    boolean getResult(TreeNode pRoot,TreeNode qRoot) {
        if (pRoot == null && qRoot == null) {
            return true;
        }
        if (pRoot == null || qRoot == null) {
            return false;
        }
        return pRoot.val == qRoot.val && getResult(pRoot.left,qRoot.right) && getResult(pRoot.right,qRoot.left);
    }
}
~~~

###### 5.2、迭代

~~~java
class Solution {
    // 转换为两个二叉树的
    public boolean isSymmetric(TreeNode root) {
        return check(root,root);
    }
    boolean check(TreeNode pRoot,TreeNode qRoot) {
        // 初始化一个队列；将两个节点都放进去  每层弹出控制下一层加入的个数；提前满足条件提前退出
        Queue<TreeNode> q = new LinkedList<TreeNode>();
        q.offer(u);
        q.offer(v);
        while(!q.isEmpty()) {
            pRoot = q.poll();
            qRoot = q.poll();
            if (pRoot == null && qRoot == null) {
                continue;
            }
            if ((pRoot == null || qRoot == null) || (pRoot.val != qRoot.val)) {
                return false;
            }
            
            q.offer(u.left);
            q.offer(v.right);

            q.offer(u.right);
            q.offer(v.left);
        }
    }
}
~~~

##### 6、二叉树锯齿形遍历

~~~java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
            //初始化结果数组
            //需要一个既能进去又能出去的数据结构,维护每一层遍历次数
            List<List<Integer>> result = new ArrayList<>();
            if (root == null) {
                return result;
            }
            Queue<TreeNode> queue = new LinkedList();
            queue.offer(root);
            int level = 0;
            while (!queue.isEmpty()) {
                LinkedList<Integer> list = new LinkedList<>();
                int size = queue.size();
                for (int i = 0; i < size; i++) {
                    TreeNode treeNode = queue.poll();
                    if (level %2 == 0) {
                        list.add(treeNode.val);
                    } else {
                        list.push(treeNode.val); 
                    }
                    //本层完了以后，加入下一层的节点，为下次遍历作准备
                    if (treeNode.left != null) {
                        queue.offer(treeNode.left);
                    }
                    if (treeNode.right != null) {
                        queue.offer(treeNode.right);
                    }
                }
                result.add(list);
                level++;
            }
            return result;
        }
}
~~~

##### 7、二叉树每一行的最大值

~~~java
    public List<Integer> largestValues(TreeNode root) {
            List<Integer> result = new ArrayList<>();
            if (root == null) {
                return result;
            }
            Queue<TreeNode> queue = new LinkedList();
            queue.offer(root);
            while (!queue.isEmpty()) {
                List<Integer> list = new ArrayList<>();
                int size = queue.size();
                int max = Integer.MIN_VALUE;
                for (int i = 0; i < size; i++) {
                    TreeNode treeNode = queue.poll();
                    max = Math.max(max,treeNode.val);
                    //本层完了以后，加入下一层的节点，为下次遍历作准备
                    if (treeNode.left != null) {
                        queue.offer(treeNode.left);
                    }
                    if (treeNode.right != null) {
                        queue.offer(treeNode.right);
                    }
                }
                result.add(max);
            }
            return result;
    }
~~~

##### 8、N 叉树的层序遍历(未完待续)

~~~java
class Solution {
    class Node {
    public int val;
    public List<Node> children;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, List<Node> _children) {
        val = _val;
        children = _children;
    }
    };
    public List<List<Integer>> levelOrder(Node root) {
            List<List<Integer>> result = new ArrayList<>();
            if (root == null) {
                return result;
            }
            Queue<Node> queue = new LinkedList();
            queue.offer(root);
            while (!queue.isEmpty()) {
                List<Integer> list = new ArrayList<>();
                int size = queue.size();
                for (int i = 0; i < size; i++) {
                    Node node = queue.poll();
                    if (node != null) {
                        list.add(node.val);
                        if (node.children != null && node.children.size() > 0) {
                           queue.addAll(node.children);
                        }
                    }
                }
                result.add(list);
            }
            return result;
        }
}
~~~

#### 2、深度优先

##### 1、深度优先搜索

~~~java
// 三种遍历方式的叫法根绝什么时候考虑根节点来决定
public void dfs(TreeNode node) {
    if (node == null) {
        return;
    }
    System.out.println(node.val);
    dfs(node.left);
    dfs(node.right);
}

// 采用栈的方式来实现二叉树前序遍历
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        if (root == null) {
            return res;
        }

        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        TreeNode node = root;
        // 这个是进行迭代的条件
        while (!stack.isEmpty() || node != null) {
            while (node != null) {
                res.add(node.val);
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            node = node.right;
        }
        return res;
    }
}

// 采用栈的方式来实现二叉树中序遍历
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        // 这个是进行迭代的条件
        while (!stack.isEmpty() || node != null) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            res.add(node.val);            
            node = node.right;
        }
        return res;
    }
}

// 采用栈的方式来实现二叉树中序遍历
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        // 这个是进行迭代的条件
        while (!stack.isEmpty() || node != null) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            res.add(node.val);            
            node = node.right;
        }
        return res;
    }
}

// 采用栈的方式来实现二叉树后序遍历
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        if (root == null) {
            return res;
        }

        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        TreeNode prev = null;
        while (root != null || !stack.isEmpty()) {
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
            if (root.right == null || root.right == prev) {
                res.add(root.val);
                prev = root;
                root = null;
            } else {
                stack.push(root);
                root = root.right;
            }
        }
        return res;
    }
}
~~~

> 从下往上：
>
> 1. 把值从下往上传
> 2. 当前递归层利用 subproblem 传上来的值计算当前层的新值并返回
> 3. 一定有返回值
>
> 基本步骤
>
> - Base Case
> - 向子问题要答案
> - 利用子问题答案构建当前问题的答案
> - 若有必要、做一些额外操作
> - 返回答案（给父问题）

##### 2、二叉树最大路径和

~~~java
// 递归返回的一般都不是当前需要的答案
int max = Integer.MIN_VALUE
public int maxPathSum(TreeNode root) {
    dfs(root);
    return max;
}
private int dfs(TreeNode node) {
    if (node == null) {
        return 0;
    }
    // 根绝子问题构建当前递归层答案
    int left = dfs(left) < 0 ? 0 : left;
    int right = dfs(right) < 0 ? 0 : right;
    max = Math.max(max,left + right + node.val);// 全局最大值的判断
    // 返回答案给父问题
    return Math.max(left + node.val,right + node.val);
}
~~~

##### 3、从上到下的dfs

> - Base Case
> - 利用父问题传下来的值做一些计算
> - 若有必要，做一些额外操作
> - 把值传下去给子问题继续做递归

##### 4、根节点和叶子节点组成的数字

~~~java
class Solution {
    int sum = 0;
    public int sumNumbers(TreeNode root) {
        if (root == null) {
            return 0;
        }
        dfs(root,0);
        return sum;
    }
    public void dfs(TreeNode root,int num) {
        num = num * 10 + root.val;
        if (root.left == null && root.right == null) {
            sum+=num;
            return;
        }
        if (root.left != null) {
            dfs(root.left,num);
        }
        if (root.right != null) {
            dfs(root.right,num);
        }
    }
}
~~~

##### 5、验证二叉搜索树

~~~java
class Solution {
private long min = Long.MIN_VALUE; 
public boolean isValidBST(TreeNode root) {
        Stack < TreeNode > stack = new Stack < > ();
        TreeNode curr = root;
        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            curr = stack.pop();
            if (min >= curr.val) {
                return false;
            }
            min = curr.val;
            curr = curr.right;
        }
        return true;
}
}
~~~

6、平衡二叉树

~~~java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        } else {
            return Math.abs(height(root.left) - height(root.right)) <= 1 && isBalanced(root.left) && isBalanced(root.right);
        }
    }

    public int height(TreeNode root) {
        if (root == null) {
            return 0;
        } else {
            return Math.max(height(root.left), height(root.right)) + 1;
        }
    }
}
~~~

##### 7、路径总和

~~~java
// 利用普通的 dfs 来进行解题   
class solution {
    List<List<Integer>> ret = new LinkedList<List<Integer>>;
    Deque<Integer> path = new LinkedList<Integer>;
    public List<List<Integer>> pathSum(TreeNode root,int sum) {
        dfs(root,sum);
        return ret;
    }
    public void dfs(TreeNode root,int sum) {
        if (root == null) {
            return;
        }
        path.offerLast(root.val);
        sum -= root.val;
        if (root.left == null || root.right == null && sum == 0) {
            ret.add(new LinkedList<Integer>);
        }
        dfs(root.left,sum);
        dfs(root.right,sum);
        path.poolLast(root.val);
    }
}

~~~

##### 8、二叉树的最近公共祖先（是两个节点的祖先且深度尽可能大）

~~~java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q)
{
    if (root == null || root == p || root == q) {
        return root;
    }
    TreeNode left = lowestCommonAncestor(root.left,p,q);
    TreeNode right = lowestCommonAncestor(root.right,p,q);
    if (left != null && right != null) {
        return root;
    }
    return left == null ? right : left;
}
}
~~~

##### 9、删除二叉树中的节点

~~~java
// 备注：二叉搜索树中序遍历是递增的序列
// 中序遍历前一个节点
public int predecessor(TreeNode root) {
  root = root.left;
  while (root.right != null) root = root.right;
  return root;
} 
// 中序遍历后一个节点
public int successor(TreeNode root) {
  root = root.right;
  while (root.left != null) root = root.left;
  return root;
} 

//删除分情况
// 叶子结点直接删除
// 不是拥有右节点 该节点可以由该节点的后继节点进行替代 从后继节点向下操作以删除后继节点
// 不是拥有左节点 以使用它的前驱节点进行替代，然后再递归的向下删除前驱节点

class Solution {
  /*
  One step right and then always left
  */
  public int successor(TreeNode root) {
    root = root.right;
    while (root.left != null) root = root.left;
    return root.val;
  }

  /*
  One step left and then always right
  */
  public int predecessor(TreeNode root) {
    root = root.left;
    while (root.right != null) root = root.right;
    return root.val;
  }

  public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    //  不相等的时候一直递归
    if (key > root.val) {
        root.right = deleteNode(root.right, key);
    } 
    else if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } 
    else {
      if (root.left == null && root.right == null) {
          root = null;
      } 
      else if (root.right != null) {
        root.val = successor(root);
        root.right = deleteNode(root.right, root.val);
      }
      else {
        root.val = predecessor(root);
        root.left = deleteNode(root.left, root.val);
      }
    }
    return root;
  }
}
~~~



#### 3、填充每个节点的下一个右侧指针节点

##### 3.1、递归

~~~java
class Solution {
    public Node connect(Node root) {
        if (root == null) {
            return root;
        }
        
        // 初始化队列同时将第一层节点加入队列中，即根节点
        Queue<Node> queue = new LinkedList<Node>(); 
        queue.add(root);
        
        // 外层的 while 循环迭代的是层数
        while (!queue.isEmpty()) {
            
            // 记录当前队列大小
            int size = queue.size();
            
            // 遍历这一层的所有节点
            for (int i = 0; i < size; i++) {
                
                // 从队首取出元素
                Node node = queue.poll();
                
                // 连接 最右边一个节点不进行连接
                if (i < size - 1) {
                    node.next = queue.peek();
                }
                
                // 拓展下一层节点
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
        }
        
        // 返回根节点
        return root;
    }
}
~~~

##### 3.2、迭代

~~~java
class Solution {
    public Node connect(Node root) {
        if (root == null) {
            return root;
        }
        
        // 从根节点开始
        Node leftmost = root;
        
        while (leftmost.left != null) {
            
            // 遍历这一层节点组织成的链表，为下一层的节点更新 next 指针【最左边指针一直往后移动，上一层的数量控制下一层的移动】
            Node head = leftmost;
            
            while (head != null) {
                
                // CONNECTION 1
                head.left.next = head.right;
                
                // CONNECTION 2
                if (head.next != null) {
                    head.right.next = head.next.left;
                }
                
                // 指针向后移动
                head = head.next;
            }
            
            // 去下一层的最左的节点
            leftmost = leftmost.left;
        }
~~~

### 7、动态规划

#### 1 D

##### 1、子序列问题

> 每一个 Node 代表一个状态 -》（index,current_subset）

~~~java
 1. 定义子问题状态
 2. 初始化初始状态
 3. 回调 dfs
 具体逻辑：
     a. 用下一个状态更新当前状态
     b. 向“下”走 -》调用 dfs
     c. 恢复成原有状态，保证每一个子问题进来都是一样的
 
 public List<List<Integer>> subsets(int[] nums) {
     List<List<Integer>> res = new ArrayList<>();
     dfs(res,nums,new ArrayList<>(),0);
     return res;
 }
 public void dfs(List<List<Integer>> res,int[] nums,List<Integer> cur,int index){
     // Base case
     if (index > nums.length) {
         res.add(new ArrayList<>(cur));
         return;
     }
     // 用下一个状态更新当前状态
     cur.add(num[index]);
     dfs(res,nums,cur,index+1);
     // 恢复成原有状态，保证每一个子问题进来都是一样的
     cur.remove(cur.size() - 1);
     dfs(res,nums,cur,index+1);
 } 
~~~

##### 2、Word Break

~~~java
dfs(state):
1. Base Case : i == 0; return true;
2. if (memo[len]) 已经计算过直接返回
3. 对于每一个 [0,i) 的子问题
   a. 去向子问题要答案 call y = dfs(j)
   b. 如果 y == true && word.substring(j,len) --> true
4. memo[len] = ans for step 3
5. return memo[len]
Boolean[] memo; 
    public boolean wordBreak(String s, List<String> wordDict) {
        int len = s.length();
        memo = new Boolean[len + 1];
        return dfs(s,len,new HashSet(wordDict));
    }
    private boolean dfs(String s,int len,Set<String> wordDict) {
        if (len == 0) {
            return true;
        }
        if (memo[len] != null) {
            return memo[len];
        }
        memo[len] = false;
        // 根据实际情况考虑 是 0 还是 1 右边是小于还是小于等于  右边为空没有任何实际意义
        for (int i = 0;i < len;i++) {
            boolean right = wordDict.contains(s.substring(i,len));
            if (!right) {
                continue;
            }
            boolean left = dfs(s,i,dict);
            if (left) {
                memo[len] = true;
                break;
            }
        }
        return mem[len];
    }
~~~

##### 3、唯一的二叉搜索树

~~~java
class Solution {
    Integer[] memo;
    public int numTrees(int n) {
        memo = new Integer[n+1];
        return dfs(n);
    }
    private int dfs(int n) {
        if (n <= 1) {
            return 1;
        }
        if (memo[n] != null) {
            return memo[n];
        }
        int res = 0;
        // 必须要有一个根节点  所以必须从1  开始  左右节点都可以为空  所以可以采用闭区间
        for (int i = 1;i<=n;i++) {
            res += dfs(i-1) * dfs(n-i);    
            // dfs 后边的变量多少 根据你你后边需要计算的来决定 根本就是 dfs(i)  里边 i 的 取法取决于具体问题
        }
        memo[n] = res;
        return res;
    }
}
~~~

##### 4、解码方法

~~~java
class Solution {
    Integer[] memo;
    public int numDecodings(String s) {
        int len = s.length();
        memo = new Integer[len + 1];
        return dfs(s,len);
    }
    private int dfs(String s,int n) {
        if (n == 0) {
            return 1;
        }
        if (n == 1) {
            return s.charAt(0) == '0' ? 0 : 1;
        }
        if (memo[n] != null) {
            return memo[n];
        }
        int res = 0;
        char x = s.charAt(n - 1);
        char y = s.charAt(n - 2);
        if (x != '0') {
            res += dfs(s,n-1);
        }
        int yx = (y - '0') * 10 + (x - '0');
        if (yx >= 10 && yx <= 26) {
            res += dfs(s,n-2);
        }
        memo[n] = res;
        return res;
    }
}
~~~

#### 2 D

> - （row,col）
> - 2 个 1D Arrays
> - 1D Arrays + K
> - 1D Arrays -》 2D state

##### 1、有多少种唯一的路径

~~~java
class Solution {
    Integer[][] memo;
    int m,n;
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        m = obstacleGrid.length;
        n = obstacleGrid[0].length;
        memo = new Integer[m][n];
        // 根据题目要求  左边或者上边就是当前子问题的答案  到最后一个终点的时候
        return dfs (obstacleGrid,m-1,n-1);
    }
    private int dfs (int[][] obstacleGrid,int i,int j){
        if(i < 0 || i >= m || j < 0 || j >= n || obstacleGrid[i][j] == 1){
            return 0;
        }
        if (i == 0 && j == 0) {
            return 1;
        }
        if (memo[i][j] != null) {
            return memo[i][j];
        }
        int res = 0;
        res += dfs(obstacleGrid,i - 1,j);
        res += dfs(obstacleGrid,i,j - 1);
        return memo[i][j] = res;
    }
}
~~~

~~~java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m = obstacleGrid.length;
        int n = obstacleGrid[0].length;
        Integer[][] memo = new Integer[m+1][n+1];
        memo[1][1] = 1;
        for (int i = 1;i <=m;i++){
            for (int j = 1;j <=n;j++) {
                if (obstacleGrid[i - 1][j - 1] == 1) {
                   memo[i][j] = 0; 
                } else {
                   memo[i][j] +=  memo[i-1][j] + memo[i][j-1];
                }
            }
        }
        return memo[m][n];
    }
}
~~~

##### 2、最长公共子序列

> **df**s(t1,t2,i,j)**
>     **1. Base case : (i,j)越界--》return 0**
>     **2. if memo[i][j] != null --》return memo[i][j]**
>     **3. 问子问题要答案**
>         **a. t1[i] = t2[j]--》result = dfs(t1,t2,i-1,j-1)**
>         **b. t1[i] != t2[j]--》result = max(dfs(t1,t2,i,j-1),dfs(t1,t2,i-1,j))**
>
> 4. 更新 memo 并且返回**

~~~java
    class Solution {
        Integer[][] memo;
        int m, n;

        public int uniquePathsWithObstacles(String text1, String text2) {
            m = text1.length();
            n = text2.length();
            memo = new Integer[m][n];
            // 根据题目要求  左边或者上边就是当前子问题的答案  到最后一个终点的时候
            return dfs(text1, text2, m, n);
        }

        private int dfs(String text1, String text2, int i, int j) {
            if (i < 0 || j < 0) {
                return 0;
            }
            if (memo[i][j] != null) {
                return memo[i][j];
            }
            int res = 0;
            if (text1.charAt(i) == text1.charAt(j)) {
                res = dfs(text1, text2, i - 1, j - 1) + 1;
            } else {
                Math.max(dfs(text1, text2, i - 1, j), dfs(text1, text2, i, j - 1));
            }
            return memo[i][j] = res;
        }
    }
~~~

##### 3、分割数组的最大值

###### 1、正常解法

~~~java
/* dfs(nums,i,j)   m 表示分割成数组的个数 n 表示剩下元素的个数
1. Base case m > n --> return -1
   m == 1 retuen sum(0..n)
2. if (memo[i][j] != null) {
    return memo[i][j];
}
3. Ask subProblems for answers
   a. 从 [1,n)，left = dfs(m-1,k),right = sum(k...n)
      result = min(max(left,right))  all the ks
4.  赋值并返回 memo[][]
*/
Integer [][] memo;
int[] prefix[];
public int spiltArray(int[] nums,int m) {
    int n = nums.length;
    memo = new Integer[m+1][n+1];
    prefix = new int[n+1];
    for (int i = 0;i < n;i++) {
        prefix[i + 1] = prefix[i] + nums[i];
    }
    return dfs(nums,m,n);
}
private int dfs(int[] nums,int m,int n) {
    if (m == 1) {
        return prefix[n];
    }
    if (m > n) {
        return -1;
    }
    if (memo[m][n] != null) {
        return memo[m][n];
    }
    int res = Integer.MAX_VALUE;  // 需要一个标识的值 最大值就用最大
    for (int k = 1;k < n;k++){
        int left = dfs(nums,m-1,k);
        if (left == -1) {
            continue;
        }
        int right = prefix[n] - prefix[k];
        int sub = Max.max(left,right);
        res = Math.min(res,sub);
    }
    res = res == Integer.MAX_VALUE ? -1 : res;
    return memo[m][n] = res;
}

~~~

###### 2、正向 DP

~~~java
/* 各自和最大值最小
1. 初始化 memo[i+1][j+1];
2. memo[1][0...n] = sum(0,n)
   m 表示分割成数组的个数  数组长度为 n
3. 使用规则
   a. for i from [2,m]  切一次 等于是
          for j from [i,n]  make sure j > i
              for k from [1,j]  make sure 至少进行切割
                  memo[i][j] = min(memo[i][j],max(memo[i-1][k],memo[1][j] - memo[1][k]));
*/
public int spiltArray(int[] nums,int m) {
    int n = nums.length;
    int[][] memo = new int[m+1][n+1];
    for (int i = 0;i < n;i++) {
        memo[1][i+1] = memo[1][i] + nums[i];
    }
    for (int i = 0;i <= m;i++) {
        for (int j = i;j <= n,j++) {
            meme[i][j] = Integer.MAX_VALUE;
            for (int k = 1;k < k;k++) {
                int sub = Math.max(memo[i-1,k],memo[1][j] - memo[1][k]);
                memo[i][j] = Math.min(memo[i][j],sub);
            }
        }
    }
    return memo[m][n];
}
~~~

##### 4、最好的买卖股票的时间

###### 1、正常解法

~~~java
/* 
dfs(nums,k,n)  nums 表示价格数组,k 表示交易次数,n 表示数组长度
 1. Base case k == 0 || n <= 1 --> return 0
 2. if (memo[i][j] != null) --> return memo[i][j];
 3. ask subproblems for answers
    a. 不做交易，直接继承前边答案 result = dfs(nums,k,n-1);
    b. 取所有交易的最大值
       for i from [1,n) 后边的表示不做任何交易
       res = max(res,dfs(k-1,i) + prices[n-1] - prices[i-1]);
 4. update memo and return result; 
 */
Integer[][] memo;
public int maxProfit(int k,int[] prices) {
    int n = prices.length;
    if (n == 0) {
       return 0; 
    }
    if (k > n/2) {
        int max = 0;
        for (int i = 0;i < n-1;i++) {
            if (prices[i] < prices[i+1]) {
                max += prices[i+1] - prices[i];
            }
        }
        return max;
    }
    memo = new Integer[k+1][n+1];
    return dfs(prices,k,n);
}
private int dfs(int[] prices,int k,int n) {
    if (k == 0 || n <= 1) {
        return 0;
    }
    if (memo[k][n] != null) {
        return memo[k][n];
    }
    int res = dfs(k,n-1);  // n-1 是继承前一个
    for (int i = 1;i < n;i++) { // k-1 是向子问题要答案
        res = Math.max(res,dfs(prices,k-1,i) + prices[n-1] - prices[k-1]);
        return memo[k][n] = res;
    }
}
~~~

###### 2、正向 DP

~~~java
public int maxProfit(int k,int[] prices) {
    int n = prices.length;
    if (n == 0) {
        return 0;
    }
    if (k > n/2) {
        int max = 0;
        for (int i = 0;i < n-1;i++) {
            if (prices[i] < prices[i+1]) {
                max += prices[i+1] - prices[i];
            }
        }
        return max;
    }
    int[][] memo = new int[k+1][n+1];
    for (int i = 1;i <= k;i++) {
        for (int j = 1;j <= n;j++) {
            for (int x = 1;x < j;x++) {
                memo[i][j] = Math.max(memo[i][j],memo[i-1][x] + prices[j-1] - prices[x-1])
            }
        }
    }
    return memo[k][n];
}
~~~

##### 5 、Stone Game

###### 1、正常解法

~~~java
/*
  dfs(piles,i,j)
  1. i == j --> return piles[i]
  2. if memo[i][j] != null--> return memo[i][j];
  3. Ask subproblems for answers
     a. result = max(piles[i+1] - dfs(i+1,j),piles[j] - dfs(i,j-1));
  4. update memo and return result
*/
Integer[][] memo;
public boolean stoneGame(int[] piles) {
    int n = piles.length;
    memo = new Integer[n][n];
    dfs(piles,0,n-1);
    return memo[0][n-1] > 0;
}
private int dfs(int[] piles,int i,int j) {
    if (i > j) {
        return 0;
    }
    if (i == j) {
        return piles[i];
    }
    if (memo[i][j] != null) {
        return memo[i][j];
    }
    int res = Math.max(piles[i] - dfs(piles,i+1,j),piles[j] - dfs(piles,i,j-1));
    return memo[i][j] = res;
}
~~~

###### 2、正向 DP

~~~java
public boolean stoneGame(int[] piles) {
    int n = piles.length;
    int[][] memo = new int[n][n];
    for (int i =0;i < n;i++) {
        memo[i][i] = piles[i];
    }
    for (int l =2;l <= n;l++) {
        for (int i = 0;i < n-1;i++) {
            int j = i + l -1;
            memo[i][j] = Math.max(piles[i] - memo[i+1][j],piles[j] - memo[i][j-1]);
        }
    }
    return memo[0][n-1] > 0;
}
~~~

##### 6、最长回文字符串

###### 1、基本解法

~~~java
/*
state :
dfs(s,i,j)
1. Base case i >= j--> return true  更新下全局变量;
2. if memo[i][j] != null --> return memo[i][j]
3. Ask subproblems for answers
   a. if s[i] == s[j] && dfs(s,i+1,j-1) --> memo[i][j] = true
      更新下全局变量
   b. else dfs(s,i+1,j),dfs(s,i,j-1);
 4. update memo and return result;
*/
Boolean[][] memo;
string lps = "";
public String longestPalindrome(String s) {
    int n = s.length();
    memo = new Boolean[n][n];
    dfs(s,0,n-1);
    return lps;
}
private boolean dfs(String s,int i,int j) {
    if (i >= j) {
        if (lps.length() < j - i + 1) {
            lps = s.substring(i,j+1);
        }
        return true;
    }
    if (memo[i][j] != null) {
        return memo[i][j];
    }
    boolean res = false;
    if (s.charAt(i) == s.charAt(j) && dfs(s,i+1,j-1)) {
        res = true;
        if (lps.length() < j - i + 1) {
            lps = s.substring(i,j+1);
        }
    } else {
        dfs(s,i+1,j);
        dfs(s,i,j-1)
    }
    return memo[i][j] = res;
}

~~~

###### 2、正向 DP

~~~java
public String longestPalindrome(String s) {
    int n = s.length();
    if (n == 0) {
        return "";
    }
    String lps = s,substring(0,1);
    boolean[][] memo = new boolean[n][n];
    for (int i = 0;i < n++) {
        memo[i][i] = true;
    }
    for (int i = 0;i < n++) {
        memo[i][i-1] = true;
    }
    for (int l = 2;l <= n;i++) {
        for (int i = 0;i< = n-1;i++) {
            int j = i + l -1;
            s.charAt(i) == s.charAt(j) && memo[i+1],[j-1];
            if (memo[i][j] && lps.length() < l) {
                lps = s.substring(i,j+1);
            }
        }
    }
    return lps;
}
~~~











