### 先移动，再更新，是一个很普遍的bug

(容易出现在图论，链表，回溯各种之中)



## 二分查找里面一定要是

left + 1, right - 1不然一定出bug

```c++
void search(int n, vector<int> nums) {
	int left = 0, right = nums.size() - 1, value = -1;//value很重要
	while (left <= right) {
		int mid = (left + right) / 2;
		if (a[mid] <= n) {
			value = mid;
			left = mid - 1;
		}
	}
}
```



## string里面+操作会增加时间

## 记住，记忆化搜索的时候，不要创建大数组，用map更省时间。

## 典中点，C++很快的写法[&](vector<int>& i, vector<int>& j){return i[1] < j[1]>};

```
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        if (intervals.size() == 1)
            return 0;
        sort(intervals.begin(), intervals.end(), [&](vector<int>& i, vector<int>& j){return i[1] < j[1];});
        int count = 1;
        int n = intervals.size();
        int last = intervals[0][1];
        for (int i = 1; i < n; i++) {
            if (intervals[i][0] >= last) {
                count++;
                last = intervals[i][1];
            }
        }
        return n - count;
    }
};
```