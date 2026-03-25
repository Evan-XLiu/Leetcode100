# 73. 矩阵置零

## 题目描述
* 给定一个 `m x n` 的矩阵，如果一个元素为 `0` ，则将其所在行和列的所有元素都设为 `0` 。请使用 **原地** 算法。

### 示例
* **示例 1：**
    * 输入：matrix = [[1,1,1],[1,0,1],[1,1,1]]
    * 输出：[[1,0,1],[0,0,0],[1,0,1]]
* **示例 2：**
    * 输入：matrix = [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
    * 输出：[[0,0,0,0],[0,4,5,0],[0,3,1,0]]

### 限制
* `m == matrix.length`
* `n == matrix[0].length`
* $1 \le m, n \le 200$
* $-2^{31} \le matrix[i][j] \le 2^{31} - 1$

### 进阶
* 一个直观的解决方案是使用 $O(mn)$ 的额外空间，但这并不是一个好的解决方案。
* 一个简单的改进方案是使用 $O(m + n)$ 的额外空间，但这仍然不是最好的解决方案。
* 你能想出一个仅使用常量空间的解决方案吗？

## 解法思路
本题的核心难点在于**如何避免在置零过程中产生“污染”**。如果在遍历矩阵时直接将行和列设为 `0`，后续遍历到这些新变成的 `0` 时，会导致原本不该置零的行和列也被错误地置零。

为了解决这个问题，我们需要在修改原矩阵之前，先“记录”下哪些行和列需要被置零。以下三种方法展示了空间复杂度从 $O(m+n)$ 到 $O(1)$ 的不断优化的过程。为了方便你对比学习，我将每种方法的文字解析和对应的代码放在了一起。

## 代码实现

### python

* **方法一：使用标记数组 (直观解法，空间 $O(m+n)$)**
    * **思路和算法：** 最直白的想法是开辟两个新数组 `row` 和 `col`。第一次遍历矩阵，如果遇到 `0`，就把对应的行标记 `row[i]` 和列标记 `col[j]` 设为 `True`。第二次遍历矩阵时，只要当前位置所在行或列被标记了，就将其设为 `0`。
    * **时间复杂度:** **O(mn)**，其中 m 和 n 分别是矩阵的行数和列数。至多只需要遍历矩阵两次。
    * **空间复杂度:** **O(m+n)**，需要分别记录每一行或每一列是否有零出现。
    ```python
    class Solution:
        def setZeroes(self, matrix: List[List[int]]) -> None:
            """
            Do not return anything, modify matrix in-place instead.
            """
            m, n = len(matrix), len(matrix[0])
            row, col = [False] * m, [False] * n

            # 第一次遍历：记录哪些行和列有 0
            for i in range(m):
                for j in range(n):
                    if matrix[i][j] == 0:
                        row[i] = col[j] = True
            
            # 第二次遍历：根据标记更新原数组
            for i in range(m):
                for j in range(n):
                    if row[i] or col[j]:
                        matrix[i][j] = 0
    ```

* **方法二：使用两个标记变量 (空间 $O(1)$)**
    * **思路和算法：** 为了将额外空间降到 $O(1)$，我们可以**借用矩阵自身的第一行和第一列来当作标记数组**。但这样做有一个问题：第一行和第一列原本的数据会被覆盖掉。因此，在借用之前，我们必须先用两个额外的变量 `flag_col0` 和 `flag_row0` 记录下第一列和第一行**原本**是否就存在 `0`。预处理完这两个变量后，就可以放心大胆地用第一行/列去记录其他位置的 `0` 了。最后，再根据这两个变量决定是否要把第一行/列全部置零。
    * **时间复杂度:** **O(mn)**，至多只需要遍历矩阵两次。
    * **空间复杂度:** **O(1)**，只需要常数空间存储两个标记变量。
    ```python
    class Solution:
        def setZeroes(self, matrix: List[List[int]]) -> None:
            m, n = len(matrix), len(matrix[0])
            
            # 1. 预处理：记录第一列和第一行原本是否有 0
            flag_col0 = any(matrix[i][0] == 0 for i in range(m))
            flag_row0 = any(matrix[0][j] == 0 for j in range(n))
            
            # 2. 使用第一行和第一列作为标记数组，记录其他行列的 0
            for i in range(1, m):
                for j in range(1, n):
                    if matrix[i][j] == 0:
                        matrix[i][0] = matrix[0][j] = 0
            
            # 3. 根据标记数组，更新其他行列的元素
            for i in range(1, m):
                for j in range(1, n):
                    if matrix[i][0] == 0 or matrix[0][j] == 0:
                        matrix[i][j] = 0
            
            # 4. 最后，根据预处理的变量更新第一列和第一行
            if flag_col0:
                for i in range(m):
                    matrix[i][0] = 0
            
            if flag_row0:
                for j in range(n):
                    matrix[0][j] = 0
    ```

* **方法三：使用一个标记变量 (极致优化，空间 $O(1)$)**
    * **思路和算法：** 还能再省一点空间吗？可以！因为第一列的第一个元素 `matrix[0][0]` 恰好也是第一行的第一个元素。我们可以让 `matrix[0][0]` 专门负责标记第一行是否有 `0`。这样，我们就只需要**一个**额外的变量 `flag_col0` 来记录第一列是否有 `0` 就可以了。
    * **注意细节：** 在第二次遍历更新矩阵时，为了防止第一列的标记值 `matrix[i][0]` 被提前改变而影响该行后续元素的判断，我们**必须从最后一行开始，倒序往上处理（即从下往上，从右往左）**。
    * **时间复杂度:** **O(mn)**，至多只需要遍历矩阵两次。
    * **空间复杂度:** **O(1)**，只需要一个额外的布尔变量。
    ```python
    class Solution:
        def setZeroes(self, matrix: List[List[int]]) -> None:
            m, n = len(matrix), len(matrix[0])
            flag_col0 = False
            
            # 1. 第一次遍历：打标记
            for i in range(m):
                # 单独检查第一列
                if matrix[i][0] == 0:
                    flag_col0 = True
                # 注意 j 从 1 开始，第一行的标记用 matrix[0][0] 记录
                for j in range(1, n):
                    if matrix[i][j] == 0:
                        matrix[i][0] = matrix[0][j] = 0
            
            # 2. 第二次遍历：根据标记置零。注意外层循环必须倒序！
            for i in range(m - 1, -1, -1):
                for j in range(1, n):
                    if matrix[i][0] == 0 or matrix[0][j] == 0:
                        matrix[i][j] = 0
                
                # 当前行处理完毕后，单独处理第一列的当前元素
                if flag_col0:
                    matrix[i][0] = 0
    ```
