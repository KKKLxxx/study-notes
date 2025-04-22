# Pow函数，递归（C语言）

```c
#include <stdio.h>

int Pow(int x,int y);

int main(void)
{
	int x,y;
	scanf("%d %d",&x,&y);
	printf("%d",Pow(x,y));
	return 0;
}

int Pow(int x,int y)
{
	if(y==0)
		return 1;
	if(y%2==0)
		return Pow(x*x,y/2);
	else
		return Pow(x*x,y/2)*x;
}
```
T(n)=O(logn)