# nextcloud

个人网盘

## 使用

如果是在 Windows 10 使用，在 docker-compose up 之前，要在 power-shell 执行以下命令：  
`$Env:COMPOSE_CONVERT_WINDOWS_PATHS=1`  

其他需要配置的是根目录下的几个 env 文件。

### db.env

只需配置 nextcloud 数据库密码 `MYSQL_PASSWORD` 和 ROOT 的密码 `MYSQL_ROOT_PASSWORD`

### nginx.env

配置你的域名 `VIRTUAL_HOST`

### ssl.env

提供证书用。