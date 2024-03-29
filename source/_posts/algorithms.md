---
title: 算法笔记，JavaScript实现
date: 2022-07-28 17:11:02
categories:
  - 算法
tags:
  - 算法
---

持续更新中...


##	链表

###	链表初识
  
    链表：类似于数组，都是有序列表，线性结构。
        不同点在于，链表中，数据单位的名称叫做“结点”，而结点和结点的分布，相对于数据的连续来说在内存中可以是离散的。


 实现一个链表，通过嵌套对象的形式实现
```js
const listInJS = {
  // 数据域
  val: 1,
  // 指针域，指向下一个结点
  next: {
    val: 2,
    next: null,
  },
};
```

```js
function ListNode(val) {
  this.val = val;
  this.next = null;
}
const node = new ListNode(1);
console.log(node); //ListNode { val: 1, next: null }
```

 添加
```js
const node2 = new ListNode(2);
// 将节点2添加到以上的链表中
node.next = node2;
console.log(node); //ListNode { val: 1, next: ListNode { val: 2, next: ListNode { val: 2, next: null } } }
```

 插入
```js
const node3 = new ListNode(3);
// 将节点3插入到节点一和二之间
node3.next = node2;
node.next = node3;
console.log(node); //ListNode { val: 1,next: ListNode { val: 3, next: ListNode { val: 2, next: null } }}
```

删除
```js
// 将node3删除 将node的next节点断开链接node3，然后指向node2
node.next = node3.next;
console.log(node); //ListNode { val: 1, next: ListNode { val: 2, next: null } }
```



##	二叉树

###	二叉树
```js
const treeNode = {
  val: "A",
  left: {
    val: "B",
    left: {
      val: "D",
    },
    right: {
      val: "E",
    },
  },
  right: {
    val: "C",
    right: {
      val: "F",
    },
  },
};
```

  先序遍历二叉树   根节点->左子树->右子树
```js
function preOrder(node) {
  if (!node) return;
  console.log(node.val);
  preOrder(node.left);
  preOrder(node.right);
}
preOrder(treeNode); // A B D E C F
```

 中序遍历二叉树   左子树->根节点->右子树

```js
function inOrder(node) {
  if (!node) return;
  inOrder(node.left);
  console.log(node.val);
  inOrder(node.right);
}
inOrder(treeNode); // D B E A F C
```
 后序遍历二叉树   左子树->右子树->根节点
```js
function postOrder(node) {
  if (!node) return;
  postOrder(node.left);
  postOrder(node.right);
  console.log(node.val);
}
postOrder(treeNode); // D E B F C A
```



##	数组的应用

###	Map的妙用
>真题描述： 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
    你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。
    示例: 给定 nums = [2, 7, 11, 15], target = 9
    因为 nums[0] + nums[1] = 2 + 7 = 9 所以返回 [0, 1]


```js
//定义一个统计时间的函数
const timer = (callback) => {
  console.time("timer");
  console.log(callback());
  console.timeEnd("timer");
};
const nums = [2, 7, 11, 15, 1, 8];
const target = 9;
```

 普通解法, 使用两层for循环，用nums[i]和nums[j]相加，若等于target, 则向result中push[i, j]

```js
const twoSum1 = (nums, target) => {
  const result = [];
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        result.push([i, j]);
      }
    }
  }
  return result;
};

timer(() => twoSum1(nums, target)); //[ [ 0, 1 ], [ 4, 5 ] ]
```

 从上面的解法来看，两层循环的时间复杂度是O(n^2)，每次都要将第一次遍历的数与后面的所有数相加与target比较

 巧用map解法空间换时间，将求和问题变成球差问题

```js
const twoSum2 = (nums, target) => {
  const map = {};
  const res = [];
  for (let i = 0; i < nums.length; i++) {
    // target-当前遍历到的值的key在map中存在，则返回[map对应的value即下标, i]
    if (map[target - nums[i]] !== undefined) {
      res.push([map[target - nums[i]], i]);
    }
    // 将元素与下标作为key：value存入map
    map[nums[i]] = i;
  }
  return res;
};
timer(() => twoSum2(nums, target)); //[ [ 0, 1 ], [ 4, 5 ] ]
```
 上述的方法将nums中的元素作为key，元素的下标作为value，这样就可以求出target-nums[i]的值，如果map中有这个值，则说明有两个元素的和等于target

 使用es6的Map来做

```js
const twoSum3 = (nums, target) => {
  const map = new Map();
  const res = [];
  for (let i = 0; i < nums.length; i++) {
    // target-当前遍历到的值的key在map中存在，则返回[map对应的value即下标, i]
    if (map.get(target - nums[i]) !== undefined) {
      res.push([map.get(target - nums[i]), i]);
    }
    // 将元素与下标作为key：value存入map
    map.set(nums[i], i);
  }
  return res;
};
timer(() => twoSum3(nums, target)); //[ [ 0, 1 ], [ 4, 5 ] ]
```
 以上使用map的解法，无论是js的对象还是es6的Map，都可以实现相同的功能，然后时间复杂度都是O(n)
 留个疑问，如果你将代码运行一下就会发现使用es6的Map会比使用js对象所需的时间少个一半左右。why？

>扁平数据转树
    例：let arr = [
      { id: 1, name: "部门1", pid: 0 },
      { id: 2, name: "部门2", pid: 1 },
      { id: 3, name: "部门3", pid: 1 },
      { id: 4, name: "部门4", pid: 3 },
      { id: 5, name: "部门5", pid: 4 },
      { id: 6, name: "部门6", pid: 9 },
    ];


```js
let arr = [
  { id: 1, name: "部门1", pid: 0 },
  { id: 2, name: "部门2", pid: 1 },
  { id: 3, name: "部门3", pid: 1 },
  { id: 4, name: "部门4", pid: 3 },
  { id: 5, name: "部门5", pid: 4 },
  { id: 6, name: "部门6", pid: 9 },
];
```
 普通递归
```js

const flatToTree = (data, pid) => {
  const result = [];
  const getChildren = (data, result, pid) => {
    data.forEach((item) => {
      if (item.pid === pid) {
        const newItem = { ...item, children: [] };
        result.push(newItem);
        getChildren(data, newItem.children, item.id);
      }
    });
  };
  getChildren(data, result, pid);
  return result;
};

timer(() => flatToTree(arr, 0));
```

 使用Map的引用避免递归
```js
const flatToTree1 = (arr) => {
  const map = {};
  const res = [];
  arr.forEach((item) => {
    if (!map[item.id]) {
      map[item.id] = { children: [] };
    }
    map[item.id] = {
      ...item,
      children: map[item.id]["children"],
    };
    if (item.pid === 0) {
      res.push(map[item.id]);
    } else {
      if (!map[item.pid]) {
        // 无关数据
        map[item.pid] = {
          children: [],
        };
      }
      map[item.pid].children.push(map[item.id]);
    }
  });
  console.log(JSON.stringify(res));
  return res;
};
timer(() => flatToTree1(arr));
```

```js
function flatToTree2(items) {
  const result = []; // 存放结果集
  const itemMap = {}; //
  for (const item of items) {
    const id = item.id;
    const pid = item.pid;
    itemMap[id] = {
      ...item,
      children: [],
    };
    if (pid === 0) {
      result.push(itemMap[id]);
    } else {
      // pid没有的时候
      if (!itemMap[pid]) {
        itemMap[pid] = {
          children: [],
        };
      }
      itemMap[pid].children.push(itemMap[id]);
    }
  }
  console.log(JSON.stringify(res));
  return result;
}
console.log(JSON.stringify(flatToTree2(arr)));
```


###	双指针法合并有序数组
>真题描述：给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。
    说明: 初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。 你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。

    示例: 输入:
        nums1 = [1,2,3,0,0,0], m = 3
        nums2 = [2,5,6], n = 3
    输出: [1,2,2,3,5,6]

 定义一个统计时间的函数

```js
const timer = (callback) => {
  console.time("timer");
  console.log(callback());
  console.timeEnd("timer");
};

const nums1 = [1, 2, 3, 0, 0, 0];
const nums2 = [2, 5, 6];
const m = 3,
  n = 3;

const merge = (nums1, nums2, m, n) => {
  // 定义两个指针的初始位置和nums1的尾部的索引
  let i = m - 1,
    j = n - 1,
    k = m + n - 1;
  while (i >= 0 && j >= 0) {
    // 将大的值丢到末尾
    if (nums1[i] > nums2[j]) {
      nums1[k] = nums1[i];
      i--;
      k--;
    } else {
      nums1[k] = nums2[j];
      j--;
      k--;
    }
  }
  // 当nums2还有剩余的时候,因为题目要求返回nums1，所以将nums2剩余的添加到nums1头部
  while (j >= 0) {
    nums1[k] = nums2[j];
    k--;
    j--;
  }
};
```

 在js中的另辟蹊径
```js
const merge2 = () => {
  nums1.splice(m, n, ...nums2);
  nums1.sort((a, b) => a - b);
};
timer(() => merge(nums1, nums2, m, n));
// timer(() => merge1(nums1, nums2, m, n));
console.log(nums1); //[ 1, 2, 2, 3, 5, 6 ]
```


###	双指针法三数求和

>真题描述：给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。
    注意：答案中不可以包含重复的三元组。

    示例： 给定数组 nums = [-1, 0, 1, 2, -1, -4]， 满足要求的三元组集合为： [ [-1, 0, 1], [-1, -1, 2] ]


 定义一个统计时间的函数
```js
const timer = (callback) => {
  console.time("timer");
  console.log(callback());
  console.timeEnd("timer");
};
```
 如果按照最传统的思路可能需要三层循环，时间复杂度为 O(n^3)，肯定是不合适的。但是使用指针之后就会大大降低时间复杂度

```js
const nums = [-1, 0, 1, 2, 2, 2, -1, -4];

const threeSum = (nums) => {
  nums = nums.sort((a, b) => a - b);
  let result = [];
  for (let i = 0; i < nums.length - 2; i++) {
    let m = i + 1,
      n = nums.length - 1;

    while (m < n) {
      if (nums[m] + nums[n] + nums[i] > 0) {
        n--;
        while (nums[n] === nums[n - 1]) {
          n--;
        }
      } else if (nums[m] + nums[n] + nums[i] < 0) {
        m++;
        while (nums[m] === nums[m + 1]) {
          m++;
        }
      } else {
        result.push([nums[m], nums[n], nums[i]]);
        m++;
        n--;
        while (nums[n] === nums[n - 1]) {
          n--;
        }
        while (nums[m] === nums[m + 1]) {
          m++;
        }
      }
    }
  }
  return result;
};
timer(() => threeSum(nums));
```


  总结一下:遇到“有序”和“数组”这两个关键字，立刻把就要想到双指针法普通双指针走不通，立刻想对撞指针！




##	字符串的应用

###	回文字符串
 判断一个字符串是否是回文字符串;
```js
const str = "yessey1";

// 使用数组中的reverse判断反转后是否一致
function isPalindrome1(str) {
  // 先反转字符串
  const reversedStr = str.split("").reverse().join("");
  // 判断反转前后是否相等
  return reversedStr === str;
}
console.log(isPalindrome1(str));

function isPalindrome2(str) {
  // 缓存字符串的长度
  const len = str.length;
  // 遍历前半部分，判断和后半部分是否对称
  for (let i = 0; i < len / 2; i++) {
    if (str[i] !== str[len - i - 1]) {
      return false;
    }
  }
  return true;
}
console.log(isPalindrome2(str));
```
>真题描述：给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。
    示例 1: 输入: "aba"
    输出: True
    示例 2:
    输入: "abca"
    输出: True
    解释: 你可以删除c字符。
    注意: 字符串只包含从 a-z 的小写字母。字符串的最大长度是50000。


```js
const validPalindrome = (str) => {
  const len = str.length;
  let j = 0,
    k = len - 1;
  while (j < k && str[j] === str[k]) {
    j++;
    k--;
  }
  // 尝试判断跳过左指针元素后字符串是否回文
  if (isPalindrome(j + 1, k)) {
    return true;
  }
  // 尝试判断跳过右指针元素后字符串是否回文
  if (isPalindrome(j, k - 1)) {
    return true;
  }
  return false;
  function isPalindrome(start, end) {
    while (start < end) {
      if (str[start] !== str[end]) return false;
      start++;
      end--;
    }
    return true;
  }
};
console.log(validPalindrome(str));
```


###	字符串匹配问题
>真题描述： 设计一个支持以下两种操作的数据结构：
    void addWord(word)
    bool search(word)
    search(word) 可以搜索文字或正则表达式字符串，字符串只包含字母 . 或 a-z 。
    . 可以表示任何一个字母。

    示例: addWord("bad")
    addWord("dad")
    addWord("mad")
    search("pad") -> false
    search("bad") -> true
    search(".ad") -> true
    search("b..") -> true
    说明:
    你可以假设所有单词都是由小写字母 a-z 组成的。


```js
const timer = (callback) => {
  console.time("timer");
  console.log(callback());
  console.timeEnd("timer");
};
```

 这个需要addWord和search两个功能，所以数据需要存在某个地方。
 分析：最简单的办法就是数组，将字符串存进数组，search的时候如果没有"."就直接用includes判断，如果有就new RegExp()

```js
class WordDictionary_ {
  constructor() {
    this.words = [];
  }
  addWord(word) {
    if (!this.words.includes(word)) this.words.push(word);
  }
  search(word) {
    if (!word.includes(".")) {
      return this.words.includes(word);
    }
    // 否则是正则表达式，要先创建正则表达式对象
    const reg = new RegExp(word);
    // 只要数组中有一个匹配正则表达式的字符串，就返回true
    return this.words.some((item) => {
      return reg.test(item);
    });
  }
}
```

 这里为了降低查找时的复杂度，我们可以考虑以字符串的长度为 key，相同长度的字符串存在一个数组中，这样可以提高我们后续定位的效率。
```js
class WordDictionary {
  constructor() {
    this.words = {};
  }
  addWord(word) {
    if (this.words[word.length]) {
      this.words[word.length].push(word);
    } else {
      // 若该字符串对应长度的数组还不存在，则先创建
      this.words[word.length] = [word];
    }
  }
  search(word) {
    // 若该字符串长度在 Map 中对应的数组根本不存在，则可判断该字符串不存在
    if (!this.words[word.length]) {
      return false;
    }
    // 缓存目标字符串的长度
    const len = word.length;
    // 如果字符串中不包含‘.’，那么一定是普通字符串
    if (!word.includes(".")) {
      // 定位到和目标字符串长度一致的字符串数组，在其中查找是否存在该字符串
      return this.words[len].includes(word);
    }
    // 否则是正则表达式，要先创建正则表达式对象
    const reg = new RegExp(word);
    // 只要数组中有一个匹配正则表达式的字符串，就返回true
    return this.words[len].some((item) => {
      return reg.test(item);
    });
  }
}

const word = new WordDictionary();
word.addWord("bav");
timer(() => word.search("b.."));
```


###	字符串与数字之间的转换问题
>真题描述：请你来实现一个 atoi 函数，使其能将字符串转换成整数。
    首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。
    当我们寻找到的第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字组合起来，作为该整数的正负号；假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成整数。
    该字符串除了有效的整数部分之后也可能会存在多余的字符，这些字符可以被忽略，它们对于函数不应该造成影响。
    注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换。
    在任何情况下，若函数不能进行有效的转换时，请返回 0。
    说明： 假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 [−2^31,  2^31 − 1]。如果数值超过这个范围，请返回  INT_MAX (2^31 − 1) 或 INT_MIN (−2^31) 。
    示例 1:
    输入: "42"
    输出: 42
    示例 2:
    输入: " -42"
    输出: -42
    解释: 第一个非空白字符为 '-', 它是一个负号。
    我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。
    示例 3: 输入: "4193 with words"
    输出: 4193
    解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。
    示例 4: 输入: "words and 987"
    输出: 0
    解释: 第一个非空字符是 'w', 但它不是数字或正、负号。 因此无法执行有效的转换。
    示例 5:
    输入: "-91283472332"
    输出: -2147483648
    解释: 数字 "-91283472332" 超过 32 位有符号整数范围。因此返回 INT_MIN (−2^31) 。


```js
const atoi = (str) => {
  const reg = /\s*([-\+]?[0-9]*).*/;
  const group = str.match(reg);
  const max = Math.pow(2, 31) - 1;
  const min = -max - 1;
  let res;
  if (group) {
    res = +group[1];
    console.log(res);
    // 出去转化失败的 如 '+' '-'
    if (isNaN(res)) {
      res = 0;
    }
  }
  if (res > max) {
    return max;
  } else if (res < min) {
    return min;
  }
  return res;
};
console.log(atoi("-123")); // -123
```

这题需要对正则比较了解
首先，\s 这个符号，意味着空字符，它可以用来匹配回车、空格、换行等空白区域，这里，它用来被匹配空格。
*这个符号，跟在其它符号后面，意味着“前面这个符号可以出现0次或多次。\s*，这里的意思就是空格出现0次或多次，都可被匹配到。
接着 () 出现了。() 圈住的内容，就是我们要捕获起来额外存储的东西。
[]中的匹配符之间是“或”的关系，也就是说只要能匹配上其中一个就行了。
这里[]中包括了-和\+，-不必说匹配的是对应字符，这个\+之所以加了一个斜杠符，是因为+本身是一个有特殊作用的正则匹配符，这里我们要让它回归+字符的本义，所以要用一个\来完成转义。
[0-9]*结合咱们前面铺陈的知识，这个就不难理解了，它的意思是 0-9 之间的整数，能匹配到0个或多个就算匹配成功。
最后的 .这个是任意字符的意思，.*用于字符串尾部匹配非数字的任意字符。我们看到.*是被排除捕获组之外的，所以说这个东西其实也不会被额外存储，它被“摘除”了。




##	链表的应用

###	链表的合并
>真题描述：将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有结点组成的。 
示例： 输入：1->2->4, 1->3->4 输出：1->1->2->3->4->4


```js
// 链表构造函数
function ListNode(val, next) {
  this.val = val;
  this.next = next || null;
}
```

```js
const l1 = new ListNode(1, new ListNode(2, new ListNode(4)));
const l2 = new ListNode(1, new ListNode(3, new ListNode(4)));

const mergeTwoList = (l1, l2) => {
  // 定义一个头结点，即空链表
  const head = new ListNode();
  let cur = head;
  while (l1 && l2) {
    if (l1.val <= l2.val) {
      cur.next = l1;
      l1 = l1.next;
    } else {
      cur.next = l2;
      l2 = l2.next;
    }
    cur = cur.next;
  }
  // 一个链表走完了之后将剩余的直接挡在cur的next里面
  cur.next = l1 !== null ? l1 : l2;
  return head.next;
};
console.log(JSON.stringify(mergeTwoList(l1, l2)));
```


###	链表的删除
>真题描述：给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。
示例 1:
输入: 1->1->2
输出: 1->2
示例 2:
输入: 1->1->2->3->3
输出: 1->2->3

```js
// 链表构造函数
function ListNode(val, next) {
  this.val = val;
  this.next = next || null;
}
```
```js
const list = new ListNode(1, new ListNode(1, new ListNode(2)));
const deleteDuplicates = (list) => {
  const cur = list;
  while (!cur && !cur.next) {
    if (cur.val === cur.next.val) {
      cur.next = cur.next.next;
    } else {
      cur = cur.next;
    }
  }
  return list;
};
console.log(deleteDuplicates(list));
```


删除问题的延伸——dummy 结点


>真题描述：给定一个排序链表，删除所有含有重复数字的结点，只保留原始链表中 没有重复出现的数字。
示例 1:
输入: 1->2->3->3->4->4->5
输出: 1->2->5
示例 2:
输入: 1->1->1->2->3
输出: 2->3



我们先来分析一下这道题和上道题有什么异同哈：相同的地方比较明显，都是删除重复元素。
不同的地方在于，楼上我们删到没有重复元素就行了，可以留个“独苗”；但现在，题干要求我们只要一个元素发生了重复，就要把它彻底从链表中干掉，一个不留。
时我们就可以用一个 dummy 结点解决这个问题。
所谓 dummy 就是人为制造出来的第一个结点的前驱结点，这样链表中所有的结点都能确保有一个前驱结点，也就都能够用同样的逻辑来处理了。


```js
const list2 = new ListNode(
  1,
  new ListNode(
    1,
    new ListNode(2, new ListNode(3, new ListNode(3, new ListNode(3))))
  )
);
const deleteAllDuplicates = (list) => {
  let dummy = new ListNode();
  // dummy节点指向链表的起始位置
  dummy.next = list;
  let cur = dummy;
  // 遍历链表
  while (cur.next?.next) {
    if (cur.next.val === cur.next.next.val) {
      let val = cur.next.val;
      //存下val，向后找到不等于val的跳出循环
      while (cur.next?.val === val) {
        cur.next = cur.next.next;
      }
    } else {
      cur = cur.next;
    }
  }
  return dummy.next;
};
console.log(deleteAllDuplicates(list2));
```


###	快慢指针
 
链表题目中，有一类会涉及到反复的遍历。
往往会涉及相对复杂的链表操作，比如反转、指定位置的删除等等。

>快慢指针——删除链表的倒数第 N 个结点
真题描述：给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。
示例： 给定一个链表: 1->2->3->4->5, 和 n = 2.
当删除了倒数第二个结点后，链表变为 1->2->3->5.
说明： 给定的 n 保证是有效的。


像上面说的，这题是属于指定位置的删除，这个时候需要两次遍历，如上题，删除倒数第n个就是删除正数length+1-n个
这个时候就需要一次遍历拿到链表长度。


```js
// 链表构造函数
function ListNode(val, next) {
  this.val = val;
  this.next = next || null;
}
```
```js
const list = new ListNode(1, new ListNode(2, new ListNode(3, new ListNode(4))));
const removeNthFromEnd = (list, n) => {
  const dummy = new ListNode();
  dummy.next = list;
  let cur = dummy;
  let count = 0;
  while (cur?.next) {
    count += 1;
    cur = cur.next;
  }
  cur = dummy;
  let current = 0;
  while (cur?.next) {
    current += 1;
    if (current === count + 1 - n) {
      cur.next = cur.next.next;
    } else {
      cur = cur.next;
    }
  }
  return dummy.next;
};
// console.log(JSON.stringify(removeNthFromEnd(list, 2)));

```
不过这种超过一次的遍历必然需要引起我们的注意，我们应该主动去思考，“如果一次遍历来解决这个问题，我可以怎么做？”，这时候，就要请双指针法来帮忙了。

```js
// 使用快慢指针，快指针先走n步，然后再快慢指针同时行动，快指针遍历完的时候慢指针此时对应的就是倒数第n个
const removeNthFromEnd2 = (list, n) => {
  const dummy = new ListNode();
  dummy.next = list;
  let slow = dummy;
  let fast = dummy;
  while (fast?.next) {
    if (n > 0) {
      fast = fast.next;
      n--;
    } else {
      fast = fast.next;
      slow = slow.next;
    }
  }
  slow.next = slow.next.next;
  return dummy;
};
console.log(JSON.stringify(removeNthFromEnd2(list, 2)));
```


###	多指针实现链表的反转
链表的反转
>真题描述：定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。
示例:
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

处理链表的本质，是处理链表结点之间的指针关系。

```js
// 链表构造函数
function ListNode(val, next) {
  this.val = val;
  this.next = next || null;
}
```

 遍历链表的时候让当前节点next指向上一个节点即可，需要注意next指向改变之后会导致丢失后续的遍历，需要一个next节点用来保存断开的后续节点
```js
const list1 = new ListNode(
  1,
  new ListNode(2, new ListNode(3, new ListNode(4, new ListNode(5))))
);

const reverseList = (list) => {
  let cur = list; //头结点
  let pre = null; //前置节点
  // while遍历链表
  while (cur) {
    let next = cur.next; //先保存next节点，保证遍历能继续进行
    cur.next = pre; //反转链表
    // 向后遍历
    pre = cur;
    cur = next;
  }
  return pre;
};
console.log(JSON.stringify(reverseList(list1)));
```

局部反转一个链表
>真题描述：反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。
说明: 1 ≤ m ≤ n ≤ 链表长度。
示例:
输入: 1->2->3->4->5->NULL, m = 2, n = 4
输出: 1->4->3->2->5->NULL

```js
const list2 = new ListNode(
  1,
  new ListNode(2, new ListNode(3, new ListNode(4, new ListNode(5))))
);
const reverseBetween = (list, m, n) => {
  let pre, cur, leftHead;
  const dummy = new ListNode();
  dummy.next = list;
  //用于遍历，最初指向dummy
  leftHead = dummy;
  // 将p定位到m前面
  for (let i = 0; i < m - 1; i++) {
    leftHead = leftHead.next;
  }
  // 缓存m前一个节点
  // start 是反转区间的第一个结点
  let start = leftHead.next;
  pre = start;
  cur = pre.next;
  for (let i = m; i < n; i++) {
    let next = cur.next; //先保存next节点，保证遍历能继续进行
    cur.next = pre; //反转链表
    // 向后遍历
    pre = cur;
    cur = next;
  }
  // 反转后将反转区间前面的片段接上去
  leftHead.next = pre;
  start.next = cur;
  return dummy.next;
};
console.log(JSON.stringify(reverseBetween(list2, 2, 4)));
```


###	环形链表
>真题描述：给定一个链表，判断链表中是否有环。
示例 1：
输入：[3,2,0,4] 输出：true
解释：链表中存在一个环


```js
// 链表构造函数
function ListNode(val, next) {
  this.val = val;
  this.next = next || null;
}
```
```js
// 定义一个环形链表
const list = new ListNode(1, new ListNode(2, new ListNode(3)));
list.next.next.next = list.next; //将最后一个节点指向头结点后一位
```

我们在遍历链表的时候添加一个flag，向后遍历的时候如果存在node.flag===true 就说明链表是环形
```js
const hasCycle = (list) => {
  while (list?.next) {
    if (list?.flag) {
      return true;
    } else {
      list.flag = true;
      list = list.next;
    }
  }
  return false;
};
// console.log(hasCycle(list));
```

环形链表衍生问题——定位环的起点
这个问题和上面的其实基本一样，在第一次判断到flag为true的时候将当前节点返回即可
```js
const detectCycle = (list) => {
  while (list?.next) {
    if (list?.flag) {
      return list;
    } else {
      list.flag = true;
      list = list.next;
    }
  }
  return null;
};
console.log(detectCycle(list));
```


这道题还有一个公认的比较经典的思路，就是用快慢指针来做：
定义慢指针 slow，快指针 fast。两者齐头并进， slow 一次走一步、fast 一次 走两步。这样如果它们是在一个有环的链表里移动，一定有相遇的时刻。




##	栈与队列

###	栈的应用
括号问题
>题目描述：给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
有效字符串需满足： 左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
注意空字符串可被认为是有效字符串。
示例 1:
输入: "()"
输出: true
示例 2:
输入: "()[]{}"
输出: true
示例 3:
输入: "(]"
输出: false
示例 4:
输入: "([)]"
输出: false
示例 5:
输入: "{[]}"
输出: true


括号问题在面试中出现频率非常高， 这类题目我们一般首选用栈来做。
为什么可以用栈做？大家想想，括号成立意味着什么？意味着对称性。
巧了，根据栈的后进先出原则，一组数据的入栈和出栈顺序刚好是对称的。比如说1、2、3、4、5、6按顺序入栈，其对应的出栈序列就是 6、5、4、3、2、1 对称关系一目了然。


```js
const isValid = (str) => {
  const stack = [];
  const leftBrackets = ["{", "[", "("];
  const leftToRight = {
    "(": ")",
    "[": "]",
    "{": "}",
  };
  for (let i = 0; i < str.length; i++) {
    if (leftBrackets.includes(str[i])) {
      stack.push(leftToRight[str[i]]);
    } else {
      if (stack.pop() !== str[i]) return false;
    }
  }
  return true;
};
// console.log(isValid("({[]})")); //true
```

每日温度问题
>题目描述: 根据每日气温列表，请重新生成一个列表，对应位置的输出是需要再等待多久温度才会升高超过该日的天数。如果之后都不会升高，请在该位置用 0 来代替。
例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。
提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的均为华氏度，都是在 [30, 100] 范围内的整数。

思路：尝试去维持一个递减栈。
```js
const dailyTemperatures = (t) => {
  const stack = [];
  const res = new Array(t.length).fill(0);
  for (let i = 0; i < t.length; i++) {
    while (stack.length && t[i] > t[stack[stack.length - 1]]) {
      // 将栈顶温度值对应的索引出栈
      const top = stack.pop();
      // 计算 当前栈顶温度值与第一个高于它的温度值 的索引差值
      res[top] = i - top;
    }
    stack.push(i);
  }
  return res;
};
console.log(dailyTemperatures([73, 72, 75, 71, 69, 72, 76, 73]));
```

最小栈问题
>题目描述：设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。
push(x) —— 将元素 x 推入栈中。
pop() —— 删除栈顶的元素。
top() —— 获取栈顶元素。
getMin() —— 检索栈中的最小元素。
示例:
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin(); --> 返回 -3.
minStack.pop();
minStack.top(); --> 返回 0.
minStack.getMin(); --> 返回 -2.

```js
// 用es6的类写也是一样的
class MinStack {
  constructor() {
    this.stack = [];
  }
  push(val) {
    this.stack.push(val);
  }
  pop() {
    return this.stack.pop();
  }
  top() {
    return this.stack[this.stack.length - 1];
  }
  getMin() {
    return Math.min(...this.stack);
  }
}

const minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
console.log(minStack.getMin());
```


###	从栈到队列的世界
如何用栈实现一个队列？
>题目描述：使用栈实现队列的下列操作：
push(x) -- 将一个元素放入队列的尾部。
pop() -- 从队列首部移除元素。
peek() -- 返回队列首部的元素。
empty() -- 返回队列是否为空。
示例: MyQueue queue = new MyQueue();
queue.push(1);
queue.push(2);
queue.peek(); // 返回 1
queue.pop(); // 返回 1
queue.empty(); // 返回 false
说明:
你只能使用标准的栈操作 -- 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。
你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。
假设所有操作都是有效的 （例如，一个空的队列不会调用 pop 或者 peek 操作）。

由题意我们无法使用push和shift来简单实现队列而是需要使用栈结构，在js中没有栈结构，我们可以使用list代替，使用push  pop length 来实现队列

我们知道队列是先进先出和栈后进先出是相反的操作，我们只需要使用两个栈，stack1和stack2，在实现队列出列之前将stack1出栈存放至stack2，stack2的pop操作就相当于队列的pop操作了 peek也是同理

```js
// 上面使用class实现过了，这里使用原型
const MyQueue = function () {
  this.stack1 = [];
  this.stack2 = [];
};

/**
 * @param {number} x
 * @return {void}
 */
MyQueue.prototype.push = function (x) {
  this.stack1.push(x);
};

/**
 * @return {number}
 */
MyQueue.prototype.pop = function () {
  // 注意，只有stack2全部出栈之后的pop操作才需要同步stack1的数据
  if (this.stack2.length === 0)
    while (this.stack1?.length) {
      this.stack2.push(this.stack1.pop());
    }
  return this.stack2.pop();
};

/**
 * @return {number}
 */
MyQueue.prototype.peek = function () {
  // 与pop同理，只是不需要出栈
  if (this.stack2.length === 0)
    while (this.stack1?.length) {
      this.stack2.push(this.stack1.pop());
    }
  return this.stack2[this.stack2.length - 1];
};

/**
 * @return {boolean}
 */
MyQueue.prototype.empty = function () {
  return !this.stack1.length && !this.stack2.length;
};
const myQueue = new MyQueue();
console.log(myQueue.empty());
```

认识双端队列
>题目描述：给定一个数组 nums 和滑动窗口的大小 k，请找出所有滑动窗口里的最大值。
示例: 输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3 输出: [3,3,5,5,6,7]
解释: 滑动窗口的位置
[1 3 -1] -3 5 3 6 7
1 [3 -1 -3] 5 3 6 7
1 3 [-1 -3 5] 3 6 7
1 3 -1 [-3 5 3] 6 7
1 3 -1 -3 [5 3 6] 7
1 3 -1 -3 5 [3 6 7]
最大值分别对应：
3 3 5 5 6 7
提示：你可以假设 k 总是有效的，在输入数组不为空的情况下，1 ≤ k ≤ 输入数组的大小。

通常可以使用双指针解决，但是使用双端队列可以降低时间复杂度
```js
const maxSlidingWindow = function (nums, k) {
  let left = 0;
  let right = k - 1;
  const res = [];
  while (right < nums.length) {
    let max = nums[left];
    for (let i = left; i < left + k; i++) {
      max = Math.max(max, nums[i]);
    }
    res.push(max);
    left++;
    right++;
  }
  return res;
};
```

O(kn)--->O(n) 使用双端队列实现一个单调递减的队列
```js
const maxSlidingWindow2 = function (nums, k) {
  const len = nums.length;
  let res = [];
  let queue = [];
  for (let i = 0; i < len; i++) {
    // 当nums[i]>= 队尾元素的时候删除掉这个元素 维持队列的单调递减
    while (nums[i] >= nums[queue[queue.length - 1]]) {
      queue.pop();
    }
    queue.push(i);
    if (i >= k - 1) {
      // 维持队列只在滑动窗口内有效
      while (queue[0] <= i - k) {
        queue.shift();
      }
      // 只有到第k-1个元素时才将第一个数添加到res;
      res.push(nums[queue[0]]);
    }
  }
  return res;
};
```



##	递归和回溯思想的应用

###	应用
>题目描述：给定一个没有重复数字的序列，返回其所有可能的全排列。
示例：   
输入: [1,2,3]
输出: [
[1,2,3],
[1,3,2],
[2,1,3],
[2,3,1],
[3,1,2],
[3,2,1]
]

```js
const permute = (nums) => {
  const res = [];
  let cur = [];
  const visited = {};

  function dfs(depth) {
    if (depth === nums.length) {
      res.push([...cur]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (!visited[nums[i]]) {
        visited[nums[i]] = 1;
        cur.push(nums[i]);
        dfs(depth + 1);
        // 每次执行到最后删除cur尾部，为下次留出空间，visited也重置
        cur.pop();
        visited[nums[i]] = 0;
      }
    }
  }
  dfs(0);
  return res;
};
console.log(permute([1, 2, 3]));
```

>题目描述：给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。
说明：解集不能包含重复的子集。
示例: 输入: nums = [1,2,3]
输出:
[
[3],
[1],
[2],
[1,2,3],
[1,3],
[2,3],
[1,2],
[]
]


```js
const subsets = function (nums) {
  const res = [];
  const len = nums.length;
  const cur = [];
  dfs(0);
  function dfs(index) {
    // 每次进入，都意味着组合内容更新了一次，故直接推入结果数组
    res.push(cur.slice());
    // 从当前数字的索引开始，遍历 nums
    for (let i = index; i < len; i++) {
      // 这是当前数字存在于组合中的情况
      cur.push(nums[i]);
      // 基于当前数字存在于组合中的情况，进一步 dfs
      dfs(i + 1);
      // 这是当前数字不存在于组合中的情况
      cur.pop();
    }
  }
  return res;
};
console.log(subsets([1, 2, 3]));
```

>题目描述：给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。
示例: 输入: n = 4, k = 2
输出:
[
[2,4],
[3,4],
[2,3],
[1,2],
[1,3],
[1,4],
]

```js
const combine = function (n, k) {
  const res = [];
  const cur = [];
  dfs(1);
  function dfs(index) {
    if (cur?.length === k) {
      res.push([...cur]);
      return;
    }
    for (let i = index; i <= n; i++) {
      cur.push(i);
      dfs(i + 1);
      cur.pop();
    }
  }
  return res;
};
console.log(combine(4, 2));
```

什么时候用递归回溯思想
看两个特征：
题目中暗示了一个或多个解，并且要求我们详尽地列举出每一个解的内容时，一定要想到 DFS、想到递归回溯。  
题目经分析后，可以转化为树形逻辑模型求解。
一个模型——树形逻辑模型；两个要点——递归式和递归边界。
树形逻辑模型的构建，关键在于找“坑位”，一个坑位就对应树中的一层，每一层的处理逻辑往往是一样的，这个逻辑就是递归式的内容。至于递归边界，要么在题目中约束得非常清楚、要么默认为“坑位”数量的边界。  

>function xxx(入参) {
  前期的变量定义、缓存等准备工作 
  // 定义路径栈
  const path = []
  // 进入 dfs
  dfs(起点) 
  // 定义 dfs
  dfs(递归参数) {
    if(到达了递归边界) {
      结合题意处理边界逻辑，往往和 path 内容有关
      return   
    }
    // 注意这里也可能不是 for，视题意决定
    for(遍历坑位的可选值) {
      path.push(当前选中值)
      处理坑位本身的相关逻辑
      path.pop()
    }
  }
}

没有思路的时候对照着这个模板想一想说不定就有了呢



##	二叉树真题归纳解读

###	二叉树遍历
先（前）序遍历迭代实现
>题目描述：给定一个二叉树，返回它的前序（先序）遍历序列。
示例: 输入: [1,null,2,3]
1   
 \   
  2   
 /  
3 
输出: [1,2,3]
进阶: 递归算法很简单，你可以通过迭代算法完成吗？

```js
function TreeNode(val, left, right) {
  this.val = val === undefined ? 0 : val;
  this.left = left === undefined ? null : left;
  this.right = right === undefined ? null : right;
}
```
不使用递归 使用迭代算法

前序遍历的规则是，先遍历根结点、然后遍历左孩子、最后遍历右孩子——这正是我们所期望的出栈序列。按道理，入栈序列和出栈序列相反，我们似乎应该按照 右->左->根 这样的顺序将结点入栈。不过需要注意的是，我们遍历的起点就是根结点，难道我们要假装没看到这个根结点、一鼓作气找到最右侧结点之后才开始进行入栈操作吗？答案当然是否定的，我们的出入栈顺序应该是这样的：  
1.将根结点入栈 
2.取出栈顶结点，将结点值 push 进结果数组 
3.若栈顶结点有右孩子，则将右孩子入栈
4.若栈顶结点有左孩子，则将左孩子入栈

```js
var preorderTraversal = function (root) {
  const stack = [root];
  const res = [];
  if (!root) {
    return res;
  }
  while (stack?.length) {
    const cur = stack.pop();
    res.push(cur.val);
    if (cur.right) {
      stack.push(cur.right);
    }
    if (cur.left) {
      stack.push(cur.left);
    }
  }
  return res;
};
```
异曲同工的后序遍历迭代实现
```js
const postorderTraversal = function (root) {
  const res = [];
  if (!root) {
    return res;
  }
  const stack = [root];
  while (stack.length) {
    const cur = stack.pop();
    res.unshift(cur.val);
    if (cur.left) {
      stack.push(cur.left);
    }
    if (cur.right) {
      stack.push(cur.right);
    }
  }
  return res;
};
```
思路清奇的中序遍历迭代实现
```js
const inorderTraversal = function (root) {
  const res = [];
  const stack = [];
  let cur = root;
  while (cur || stack.length) {
    // 这个 while 的作用是把寻找最左叶子结点的过程中，途径的所有结点都记录下来
    while (cur) {
      // 将途径的结点入栈
      stack.push(cur);
      // 继续搜索当前结点的左孩子
      cur = cur.left;
    }
    // 取出栈顶元素
    cur = stack.pop();
    // 将栈顶元素入栈
    res.push(cur.val);
    // 尝试读取 cur 结点的右孩子
    cur = cur.right;
  }
  return res;
};
```

层序遍历的衍生问题
>题目描述：给你一个二叉树，请你返回其按 层序遍历 得到的节点值。 （即逐层地，从左到右访问所有节点）。
示例： 二叉树：[3,9,20,null,null,15,7],
  3
 / \
9  20
  /  \
 15   7
返回其层次遍历结果：
[
[3],
[9,20],
[15,7]
]

```js
const levelOrder = function (root) {
  // 初始化结果数组
  const res = [];
  // 处理边界条件
  if (!root) {
    return res;
  }
  // 初始化队列
  const queue = [];
  // 队列第一个元素是根结点
  queue.push(root);
  // 当队列不为空时，反复执行以下逻辑
  while (queue.length) {
    // level 用来存储当前层的结点
    const level = [];
    // 缓存刚进入循环时的队列长度，这一步很关键，因为队列长度后面会发生改变
    const len = queue.length;
    // 循环遍历当前层级的结点
    for (let i = 0; i < len; i++) {
      // 取出队列的头部元素
      const top = queue.shift();
      // 将头部元素的值推入 level 数组
      level.push(top.val);
      // 如果当前结点有左孩子，则推入下一层级
      if (top.left) {
        queue.push(top.left);
      }
      // 如果当前结点有右孩子，则推入下一层级
      if (top.right) {
        queue.push(top.right);
      }
    }
    // 将 level 推入结果数组
    res.push(level);
  }
  // 返回结果数组
  return res;
};
```


###	翻转二叉树
>题目描述：翻转一棵二叉树。
示例：
输入：

     4
   /   \
  2     7
 / \   / \
1   3 6   9
输出：
     4
   /   \
  7     2
 / \   / \
9   6 3   1

```js
const invertTree = function (root) {
  // 定义递归边界
  if (!root) {
    return root;
  }
  // 递归交换右孩子的子结点
  let right = invertTree(root.right);
  // 递归交换左孩子的子结点
  let left = invertTree(root.left);
  // 交换当前遍历到的两个左右孩子结点
  root.left = right;
  root.right = left;
  return root;
};
```


###	二叉树搜索专题
二叉搜索树（Binary Search Tree）简称 BST，是二叉树的一种特殊形式。它有很多别名，比如排序二叉树、二叉查找树等等。

什么是二叉搜索树？
1. 是一棵空树
2. 是一棵由根结点、左子树、右子树组成的树，同时左子树和右子树都是二叉搜索树，
且左子树上所有结点的数据域都小于等于根结点的数据域，右子树上所有结点的数据域都大于等于根结点的数据域
即 `左孩子 <= 根结点 <= 右孩子`
关于二叉搜索树，大家需要掌握以下高频操作：
1. 查找数据域为某一特定值的结点
2. 插入新结点
3. 删除指定结点

```js
function TreeNode(val, left, right) {
  this.val = val === undefined ? 0 : val;
  this.left = left === undefined ? null : left;
  this.right = right === undefined ? null : right;
}
```
查找数据域为某一特定值的结点
```js
const search = (root, n) => {
  if (!root) return;
  if (root.val === n) {
    console.log("目标结点是：", root);
  } else if (root.val > n) {
    search(root.left, n);
  } else if (root.val < n) {
    search(root.right, n);
  }
};
```
插入新结点
```js
function insertIntoBST(root, n) {
  // 若 root 为空，说明当前是一个可以插入的空位
  if (!root) {
    // 用一个值为n的结点占据这个空位
    root = new TreeNode(n);
    return root;
  }

  if (root.val > n) {
    // 当前结点数据域大于n，向左查找
    root.left = insertIntoBST(root.left, n);
  } else {
    // 当前结点数据域小于n，向右查找
    root.right = insertIntoBST(root.right, n);
  }

  // 返回插入后二叉搜索树的根结点
  return root;
}
```

删除指定结点
结点删除后只需要继续保持二叉查找树的特性即可
```js
const deleteNode = (root, n) => {
  if (!root) return root;
  if (root.val === n) {
    if (!root.left && !root.right) {
      root = null;
    } else if (root.left) {
      // 为了保持二叉搜索树的特性需要将左子树的最大值替换掉当前节点
      // 因为左子树的最大值满足大于所有左子树的值小于所有右子树的值
      const maxLeft = findMax(root.left);
      // 覆盖原有值
      root.val = maxLeft.val;
      // 删除这个节点
      root.left = deleteNode(root.left, maxLeft.val);
    } else {
      // 同理右子树中的最小值同样满足
      const minRight = findMin(root.right);
      root.val = minRight.val;
      root.right = deleteNode(root.right, minRight.val);
    }
  } else if (root.val > n) {
    root.left = deleteNode(root.left, n);
  } else {
    root.right = deleteNode(root.right, n);
  }
  return root;

  function findMax(root) {
    while (root.right) {
      root = root.right;
    }
    return root;
  }

  // 寻找右子树的最小值
  function findMin(root) {
    while (root.left) {
      root = root.left;
    }
    return root;
  }
};
```


###	二叉搜索树真题
>题目描述：给定一个二叉树，判断其是否是一个有效的二叉搜索树。
假设一个二叉搜索树具有如下特征：
节点的左子树只包含小于当前节点的数。
节点的右子树只包含大于当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。

```js
var isValidBST = function (root) {
  function dfs(root, minValue, maxValue) {
    // 若是空树，则合法
    if (!root) {
      return true;
    }
    // 若右孩子不大于根结点值，或者左孩子不小于根结点值，则不合法
    if (root.val <= minValue || root.val >= maxValue) return false;
    // 左右子树必须都符合二叉搜索树的数据域大小关系
    return (
      dfs(root.left, minValue, root.val) && dfs(root.right, root.val, maxValue)
    );
  }
  // 初始化最小值和最大值为极小或极大
  return dfs(root, -Infinity, Infinity);
};
```

>题目描述：将一个按照升序排列的有序数组，转换为一棵高度平衡二叉搜索树。
本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。
示例: 给定有序数组: [-10,-3,0,5,9],
一个可能的答案是：[0,-3,9,-10,null,5]，它可以表示下面这个高度平衡二叉搜索树：


```js
var sortedArrayToBST = function (nums) {
  if (!nums.length) return null;
  function buildBST(left, right) {
    if (left > right) return null;
    const mid = Math.floor(left + (right - left) / 2);
    const cur = new TreeNode(nums[mid]);
    cur.left = buildBST(left, mid - 1);
    cur.right = buildBST(mid + 1, right);
    return cur;
  }
  return buildBST(0, nums.length - 1);
};
```


