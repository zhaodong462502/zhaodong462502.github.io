## MySql的命令连接后查询中文乱码

### 临时修改

> 在打开的会话中使用如下命令。

```cmd
SET character_set_client = utf8mb4;
SET character_set_connection = utf8mb4;
SET character_set_results = utf8mb4;
```

