##### nginx 日志中截取带user的路径（去除？后的参数）状态码 请求时间  ，

```
cat /data/nginx/logs/www.baidu.com/web/www.baidu.com-acce*|grep user|awk '{split($7, a, "?") ;print a[1],$9,$NF}' > ~/tmp.log
cat ~/tmp.log |awk '$NF>3{print $1,$2,$NF}'> cost.log
```

##### for 循环数组

```
TABLES=(tablea tableb tablec)
for ((i=0;i<3;i++))
do
	echo ${TABLES[i]}
done

for table in ${TABLES[*]}
do
    echo ${table}
done
```

##### 修改sh文件换行编码

```
vim 文件
esc :set ff=unix enter
esc + :wq
```

或 idea 的右下角 CRLF --> LF