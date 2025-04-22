# Navicat16删除注册表

1、关闭Navicat程序

2、在搜索栏中输入`regedit`，打开注册表编辑器

3、删除`计算机\HKEY_CURRENT_USER\SOFTWARE\PremiumSoft\NavicatPremium`这一整个文件夹

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220214144024934.png" alt="image-20220214144024934" style="zoom:67%;" />

4、在`计算机\HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID`目录中，查询子文件夹中含有`Info`的一项

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220214143927945.png" alt="image-20220214143927945" style="zoom:67%;" />

5、此时重新打开Navicat，应该就已经重置试用期了

6、如果无效，可以在第3步中删除整个`PremiumSoft`文件夹，并在第4步中多查看几个文件夹，看看有没有其他只含`Info`的文件夹，将这些全都删除

