# 插入排序，归并排序，快速排序（C语言）

## 插入排序

```c
#include <stdio.h>

void insertsort(int a[],int n);

int main(void)
{
	int a[6]={6,5,4,3,2,1};
	insertsort(a,6);
	for(int i=0;i<6;i++)
		printf("%d ",a[i]);
	return 0;
}

void insertsort(int a[],int n)
{
	int i,j,temp;
	for(i=1;i<n;i++)//从第2个元素开始
	{
		temp=a[i];
		j=i-1;//从该元素的前一个元素开始，由后往前依次比较
		while(temp<a[j] && j>=0)//如果这个元素小于前面的元素
		{
			a[j+1]=a[j];//将较大的元素后移
			j--;//j-1，再比较，直到temp大于某个元素或j<0
		}
		a[j+1]=temp;//此时a[j+1]为一个空位，将temp插入
	} 
}
```
T(n)=O(n²)
稳定

## 归并排序

```c
#include <stdio.h>
#include <stdlib.h>

void MergeSort(int *num, int start, int end);
void Merge(int *num, int start, int mid, int end);

int main(void)
{
    int num[10] = {5,1,8,4,7,2,3,9,0,6};
    int length = sizeof(num) / sizeof(num[0]);
    
    MergeSort(num, 0, length-1);

    for(int i=0; i<length; i++)
        printf("%d ",num[i]);
    
    return 0;
}

void MergeSort(int *num, int start, int end)
{
    int mid = start+(end-start)/2;

    if(start >= end)
        return;

    MergeSort(num, start, mid);
    MergeSort(num, mid+1, end);
	//以上为将原数组分散为两部分再次递归，直到start>=end（每部分只含一个元素）
	
    Merge(num, start, mid, end);
}

void Merge(int *num, int start, int mid, int end)
{
    int *temp = (int *)malloc((end-start+1)*sizeof(int));//申请空间来存放两个部分归并后的临时区域
    int i = start;
    int j = mid+1;
    int k = 0;

    while(i<=mid && j<=end)//将两个分散的部分合并，任意一个部分的元素全都放入临时区域后结束循环 
    {
        if(num[i]<=num[j])//如果数组的第i项（A部分的第i-start项）<=数组的第j项（B部分的第j-mid-1项）
            temp[k++] = num[i++];//临时区域的第k项放入A部分的第i-start项，之后k和i均自增1，再比较两部分的下一项 
        else
            temp[k++] = num[j++];
    }

	//上一个循环结束后，如果某部分的元素未完全放入临时区域内，则在此处将剩余部分依次放入临时区域 
    while(i<=mid)
        temp[k++] = num[i++];
    while(j<=end)
        temp[k++] = num[j++];

    //将临时区域中排序后的元素，整合到原数组中
    for(i=0;i<k;i++)
        num[start+i] = temp[i];

    free(temp);
}
```
T(n)=O(nlogn)
稳定

## 快速排序

```c
#include <stdio.h>

void quickSort(int arr[], int low, int high);
 
int main(void)
{
    int a[10]={3,1,11,5,8,2,0,9,13,81};

    quickSort(a, 0, 9);

    for(int i=0;i<10;i++)
        printf("%d ",a[i]);

    return 0;
}

void quickSort(int arr[], int low, int high)
{
    int first = low;
    int last  = high;
    int key = arr[first];//以第一个元素为标准 
    if(low >= high)
        return;
    
    while(first < last)
    {
        while(first < last && arr[last] >= key)//从后往前，直到某个元素<=key 
            last--;
        arr[first] = arr[last];//将此元素换至key的位置（前移），此时last位置为空位

        while(first < last && arr[first] <= key)//从前往后，直到某个元素>=key 
            first++;
        arr[last] = arr[first];//将此元素换至last的位置（后移），此时first位置为空位
    }
    //最终first==last 
	//循环结束后，low至high区域内，low至first-1均为小于key的元素，first+1至high均为大于key的元素 
    
    arr[first] = key;//此时的first实际为中间位置，将key放入中间位置 

	//将元素分为两个区域，再次排序，直到每个区域只有一个元素 
    quickSort(arr, low, first-1);
    quickSort(arr, first+1, high);
}
```
T(n)=O(nlogn)
不稳定