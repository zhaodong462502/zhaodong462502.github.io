## 安装elasticsearch8.0.1之后无法访问9200:Empty reply from server

> 找到安装目录下的conf/elasticsearch.yml
>
> 将 xpack.security.enable: true 改为 xpack.security.enable: false
>
> 关闭安全功能然后重启es

