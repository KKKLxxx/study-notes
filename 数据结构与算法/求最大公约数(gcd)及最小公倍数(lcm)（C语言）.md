# 求最大公约数(gcd)及最小公倍数(lcm)（C语言）

## 法一

```c
#include <stdio.h>

int gcd(int a,int b);
 
int main(void)
{
	int a,b;
	scanf("%d %d",&a,&b);
	printf("%d %d",gcd(a,b),a*b/gcd(a,b));
	return 0;
}

int gcd(int a,int b)
{
	int temp; 
	while(b)
	{ 
		temp=a%b; 
		a=b; 
		b=temp; 
	}
	return a; 
}
```
## 法二

```c
#include <stdio.h>

int gcd(int a,int b);
 
int main(void)
{
	int a,b;
	scanf("%d %d",&a,&b);
	printf("%d %d",gcd(a,b),a*b/gcd(a,b));
	return 0;
}

int gcd(int a,int b)
{
	return b?gcd(b,a%b):a;
}
```
***
1、时间复杂度T(n)=O(logn)
2、gcd(a,b,c) = gcd(a,gcd(b,c)) = gcd(gcd(a,b),c) = gcd(gcd(a,c),b)
3、最小公倍数=原来两数乘积/最大公约数

