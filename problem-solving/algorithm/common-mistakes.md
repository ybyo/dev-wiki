# Tips

while 문 등의 내부라고 하더라도, 기준이 되는 값을 증가 시킨 후 다시 배열에 넣는 것은 오류가 발생할 확률이 매우 높음

```python
class Solution:  
    def pivotIndex(self, nums: List[int]) -> int:  
        l_sum = 0  
        r_sum = 0  
        for num in nums:  
            r_sum += num  
        r_sum -= nums[0]  
        piv = 0  
        while piv < len(nums):  
            if l_sum == r_sum:  
                return piv  
            l_sum += nums[piv]  
            piv += 1 # <- 이 경우
            r_sum -= nums[piv] # <- 여기서 오류가 발생함
        return -1
```
