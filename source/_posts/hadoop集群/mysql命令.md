## mysql开启远程登录

远程登录mysql报错 Access denied for user 'root'@  

原因：没有开启远程登录，mysql默认远程登录关闭。

```
1. 先用localhost登录（进入MySQL） mysql -u root -p
   Enter password: （输入密码）
2. 执行授权命令
   mysql> grant all  on root.* to root@'%' identified by '123'; 
   Query OK, 0 rows affected (0.07 sec)
3. 退出再试： mysql> quit
4. 再试登录： mysql -u root -h 192.168.194.142 -p
```

## 查看mysql的端口

```
netstat -tnlp | grep mysql
```

