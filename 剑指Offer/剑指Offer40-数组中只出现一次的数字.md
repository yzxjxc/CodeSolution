## 题目描述
一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

## 实例

**输入**
> [92,3,43,54,92,43,2,2,54,1]

**输出**
> 3,1

## 思路以及解答

### HashMap存储出现次数
使用`hashmap`存储数字出现的次数，`key`为出现的数字，`value`为该数字出现的次数。遍历里面所有的数字，如果`hashmap`中存在，那么`value`（次数）+1，如果`hashmap`中不存在,那么`value`置为1。

遍历完成之后，需要将次数为1的数字捞出来，同样是遍历hashmap，由于只有两个满足条件，我们设置一个标识变量，初始化为1，如果找到第一个满足条件的数字，除了写入放回数组中，还需要将该标识置为2，表示接下来找的是第2个。
如果找到第2个，那么写入之后，直接·return`。

代码如下：
```java
public int[] FindNumsAppearOnce(int[] array) {
        // write code here
        Map<Integer, Integer> maps = new HashMap<>();
        int[] num = new int[2];
        if (array != null) {
            for (int n : array) {
                Integer nums = maps.get(n);
                if (nums == null) {
                    // map中不存在
                    maps.put(n, 1);
                } else {
                    // map中已经存在
                    maps.put(n, nums + 1);
                }
            }
        }
        int index = 1;
        for (Map.Entry<Integer, Integer> entry : maps.entrySet()) {
            if (entry.getValue() == 1) {
                if (index == 1) {
                    num[0] = entry.getKey();
                    index++;
                } else {
                    num[1] = entry.getKey();
                }
            }
        }
        if (num[0] > num[1]) {
            int temp = num[0];
            num[0] = num[1];
            num[1] = temp;
        }
        return num;
    }
```

### 位运算
首先需要储备的位运算知识，异或是指二进制中，一个位上的数如果相同结果就是0，不同则结果是1.也就是如果一个数的最低位是0，另一个数的最低位是0，那么异或结果的最低位是0；如果一个数的最低位是0，另一个数的最低位是1，那么异或结果的最低位是1。

- 异或操作可以交换，不影响结果：A\^B\^C = A\^B\^C
- A\^A=0,任何一个数异或自身，等于0，因为所有位都相同
- A\^0 = A，任何一个数异或0，等于自身，因为所有位如果和0不同，就是1，也就是保留了自身的数值

假设里面出现一次的两个元素为`A`和`B`，初始化异或结果`res`为0，遍历数组里面所有的数，都进行异或操作，则最后结果`res = A^B`。

但是我们拿到这个`A`和`B`异或之后的结果，怎么区分呢？

有一种巧妙的思路，因为`A`和`B`的某一位不同才会在结果中出现`1`，说明在那一位上，`A`和`B`中肯定有一个是`0`，有一个是`1`。

那我们取出异或结果`res`最低位的1，假设这个数值是`temp`（temp只有一个位是1，也就是A和B最后不同的位）

遍历数组中的元素，和`temp`进行与操作，如果和`temp`相与，不等于0。说明这个元素的该位上也是1。那就说明它很有可能就是`A`和`B`中的一个。

只是有可能，其他的数也有可能该位上为`1`。但是符合这种情况的，其他数肯定出现两次，而`A`和`B`只可能有一个符合，并且只有可能出现一次`A`或者`B`。

凡是符合和`temp`相与,结果不为0的，我们进行异或操作。
也就是可能出现，`res1 = B^C^D^C^D...^E^E^B`或者`res1 = A^C^D^C^D...^E^E`。
上面的式子可以视为`res1 = B`或者`res1 = A`

这样操作下来，我们就知道了`res1`肯定是其中一个只出现一次的数（`A`或者`B`）,而同时上面计算出了`res = A^B`,也就是可以通过`res1^res`求出另一个数。

代码如下：
```java
 public int[] FindNumsAppearOnce (int[] array) {
        // write code here
        // A和B异或的结果
        int[] num = new int[2];
        int res = 0;
        for (int val : array) {
            res ^= val;
        }
        // temp保存了两个数最后一个不同的位
        int temp = res & (-res);
        // 保存和最后一个不同的位异或的结果
        int res1 = 0;
        for (int val : array) {
            // 不等于0说明可能是其中一个数，至少排除了另外一个数
            if ((temp & val) != 0) {
                res1 ^= val;
            }
        }
        // 由于其他满足条件的数字都出现两次，所以结果肯定就是只出现一次的数
        num[0] = res1;
        // 求出另外一个数
        num[1] = res ^ res1;
        if (num[0] > num[1]) {
            int tem = num[0];
            num[0] = num[1];
            num[1] = tem;
        }
        return num;
    }
```

**今日感想：**
位运算真的很强大！！！