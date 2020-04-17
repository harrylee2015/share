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
