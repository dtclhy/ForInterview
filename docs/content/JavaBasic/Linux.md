查看进程：ps -aux | grep ***

杀死进程：kill  9 pid

交集：sort a.txt b.txt | uniq -d

并集：sort a.txt b.txt | uniq

差集：sort a.txt b.txt b.txt | uniq -u

uniq为删除文件中重复的行，得到文件中唯一的行，参数-d 表示的是输出出现次数大于1的内容；参数-u表示的是输出出现次数为1的内容；那么对于上述的求交集并集差集的命令做如下的解释：

sort a.txt b.txt | uniq -d：将两个文件进行排序，uniq使得两个文件中的内容为唯一的，使用-d输出两个文件中次数大于1的内容，即是得到交集

sort a.txt b.txt | uniq ：将两个文件进行排序，uniq使得两个文件中的内容为唯一的，即可得到两个文件的并集

sort a.txt b.txt b.txt | uniq -u：将两个文件排序，最后输出a.txt b.txt b.txt文件中只出现过一次的内容，因为有两个b.txt所以只会输出只在a.txt出现过一次的内容(b.txt的内容至少出现两次)，即是a.txt-b.txt差集；对于b.txt-a.txt同理。

