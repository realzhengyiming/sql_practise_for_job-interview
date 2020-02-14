# python 相关

包括问到的算法实现或者笔试题之类的

## https://sort.hust.cc/6.quicksort 这儿直接看动图多语言版本的10大排序算法实现

## https://github.com/hustcc/JS-Sorting-Algorithm

（1）合并两个有序数组并且排序（无序也是可以的）

```python
def merge(l1, l2):
    res = []
    while len(l1) >0 and len(l2) > 0 :
        if l1[0] < l2[0]:
            res.append(l1[0])
            del l1[0]
        else:
            res.append(l2[0])
            del l2[0]
    res.extend(l1)
    res.extend(l2)
    return res

# 上面使用del来删掉原来的0号元素，也可以不删掉的，改用下标加一来迭代
# 无需也是一样的，这个其实就是归并排序中的排序merge合并，然后这儿可以直接 改回归并排序算法中的
def merge(l1,l2):
    res = []
    i,j = 0,0
	while i<len(l1) and j<len(l2):
        if l1[i]<=l2[j]:  # 相等的情况也可以处理了
            result.append(l1[i])  
            i+=1  
        else:
            result.append(l2[j])
            j+= 1
    res += l1[i:]  # 因为每个都是排好序的，后面的就可以直接加入进来
    res += l2[j:]  #
	return res

def merge_sort(lists):
    if len(lists)<=1:
        return lists
    halflen = len(lists)/2
    left = merge_sort[:halflen]
    right = merge_sort[halflen:]  # 这儿开始递归   （就是套娃....)
    result = merge(left,right)
    return result

if __name__="__main__":
    result = merge_sort([2,3,1,9,6,5,3])
    print(result)
    
# 归并排序两个任务，这连个都是分而治之的思维。
## 一个任务就是不停的把一个列表分成两段，
## 另一个任务是把分成两段的列表进行合并排序后返回！



干脆顺便把快排也复习一下：
快排的思路是，  
+ 1随机找一个数pivot（转轴一样）
+ 2然后给数组两边设置两个指针，目的是最终让pivot左边的数比pivot小，右边比pivot大
+ 3然后再继续左右两边的列表中迭代（套娃...这个叫做称为分区（partition）操作） 重复1，2操作，直到两个指针移到同一个数（排序完所有）
def quick_sorts(lists,left,right): # left right 是下标
    if left >= right:
        retur lists
    key = lists[left]  # 这个就是随便选了一个pivot 这儿key就是这个,这儿是个值
    low = left  # 下标小的
    high = right #下标最大的
    while left<right:
        while left<right and lists[right]>=key：  # 先移动右边的指针,右下标遇到左下标 或者 右边的值比pivot值小的时候，说明右边部分完成比pivot大
        	right -= 1  # 右边的指针右移动
        lists[left] = lists[right]   # 右边那个比pivot小的值作为下一个pivot，给lists【left]暂存
        while left<right and lists[left]<= key:   # 同上这次找到
            left+=1
        lists[right] = lists[left]
    lists[right] = key
    quick_sort(lists,low,left-1)  # 考试套娃....
    quick_sort(lists,left+1,high) # 并且不需要返回，已经把顺序替换了
    return lists
    
# 这个感觉变不过去
```



# 

