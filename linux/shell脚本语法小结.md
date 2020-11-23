# shell脚本编写注意事项

## 前言
要想真正会用linux,shell脚本编写是必备的一样技能。

## sed命令中的匹配

命令|解释
---|---
sed -i 's/old/new/g' text.txt |在text.tx中将old替换为new，s为开头，g为结束 , -i 为真正修改文件，不加为预修改
sed -i '2s/old/new/g' text.txt|在text.txt中将第2行的old替换为new，s为开头，g为结束
sed 's/^/&test/g' text.txt|在text.txt中每一行开头都添加test
sed  -i '/^test/s/test//g' text.txt|在text.txt中删除以test开头的字符串
sed 's/$/&test/g' text.txt|在text.txt中每一行结尾都添加test
sed 's/,/\n/g' text.txt|把text.txt中的每一个逗号都替换成换行
sed '/^test/d' text.txt|删除text.txt中以test开头的数据
sed  -i '/^test/s/test//g' text.txt|在text.txt中删除以test开头的字符串
sed -n '1p,$p' text.t|将text.txt中第一行，最后行打印
sed -n '1,$p' text.t|将text.txt中第一行到最后行打印

##  awk命令

命令|解释
---|---
awk -Ftest '{print $2}' test|把text.txt中以test做切割，打印切割后第二列的内容

## 数值类比较

 比较符|说明
 ----|----
 -eq | equal
-ne | not equal
-lt | less than
-le | less than or equal
-gt | greater than
-ge | greater than or equal

## eval命令

  当我们在命令行前加上eval时，shell就会在执行命令之前扫描它两次，eval命令首先对先扫描命令行中所有的命令进行置换，然后再执行该命令。该命令使用余那些一次扫描无法实现其功能的变量。


命令|解释
---|---
myfile="cat file";eval $myfile|打印file文件内容
eval "cat <<EOF $(<chain33/chain33.toml) EOF" > tmp.toml | 替换配置文件变量 chain33.toml(chain33.toml中有一些$变量)



## **[shell脚本语法备忘录](https://devhints.io/bash)**
