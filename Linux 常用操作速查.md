# 零、业务常用操作
```shell
# 查找目录下whl包 除了pip-build-deps 并 cp到目标路径
find ./ -name` `"*.whl"` `-not -path` `"*/pip-build-deps/*"
find ./ -name` `"*.whl"` `-not -path` `"*/pip-build-deps/*"` `-exec cp {} /target/path/ \;

# 命令放到后台执行 并查看日志
nohup cmd >> cmd.log` `2``>&``1` `&
jobs -l
tail -f cmd.log

# 同步文件
rsync -avPh --delete user@IP dir tar_dir

# 网络挂盘 自启动`
vim /etc/fstab`
10.7``.``67.203``:/data/public_backup /``public` `nfs defaults,_netdev,noatime` `0` `0`

# 杀进程` 
pkill -f -15 `'serve'` `&& sleep` `3` `&& pgrep -f` `'serve'` `&& pkill -f -`9` `'serve'`
pkill -f -15` `'VLLM'` `&& sleep` `3` `&& pgrep -f` `'VLLM'` `&& pkill -f `9` `'VLLM'`
```



### 一、 基础操作必备（文件与权限）
| 类别           | 常用命令                                                     | 说明                                                              |
| ------------ | -------------------------------------------------------- | --------------------------------------------------------------- |
| **文件管理**     | `ls -la`                                                 | 列出所有文件（包括隐藏），显示权限详情。                                            |
|              | `cd ~` / `cd -`                                          | 回家目录 / 回到上一个目录。                                                 |
|              | `cp -r src dst`                                          | 递归复制目录。                                                         |
|              | `mv src dst`                                             | 移动或重命名。                                                         |
|              | `rm -rf dir`                                             | 危险：强制递归删除（无确认）。                                                 |
| **文本查看**     | `cat` / `less`                                           | 查看全文（小文件） / 分页浏览（大文件，按q退出）。                                     |
|              | `head -n 10` / `tail -n 20`                              | 查看文件头/尾10行。                                                     |
|              | `tail -f app.log`                                        | 实时追踪日志更新（调试必备）。                                                 |
| **权限管理**     | `chmod 755 file`                                         | 设置权限（所有者rwx，组和其他r-x）。                                           |
|              | `chown user:group file`                                  | 更改文件所属用户和组。                                                     |
| **进程管理**     | `ps aux`                                                 | 查看所有进程详情（快照）。                                                   |
|              | `top` 或 `htop`                                           | 实时进程监控（`htop`更友好，需安装）。                                          |
|              | `kill -9 PID`                                            | 强制杀死指定PID的进程。                                                   |
| **网络**       | `netstat -tuln`                                          | 查看本机监听端口（`ss -tuln`更现代）。                                        |
|              | `curl ifconfig.me`                                       | 快速查看公网IP。                                                       |
| **存储与挂载**    | df -hT                                                   | # 查看磁盘挂载、使用率、文件系统类型                                             |
|              | du -sh /home/*                                           | 查看/home下每个用户占用了多少空间                                             |
|              | du -h --max-depth=1 /var \| sort -hr                     | 查看 /var 下哪些子目录最大（深度为1层）                                         |
|              | lsblk                                                    | # 树形查看所有块设备                                                     |
|              | blkid /dev/sdb1                                          | 获取磁盘UUID                                                        |
|              | UUID=123-456 /mnt/data ext4 defaults 0 2                 | 开机自动挂载（`/etc/fstab` 文件）写入 /etc/fstab 末尾                         |
|              | mount /dev/sdb1 /mnt/data                                | 把设备（比如 `/dev/sdb1`，即第二块硬盘的第一个分区）与一个空目录（挂载点，如 `/mnt/data`）关联起来。  |
|              | mount -t nfs IP:/dir /tar_dir                            | 网络nfs挂载                                                         |
|              | umount /mnt/data                                         | 卸载（注意：不能在挂载点目录内执行卸载）：                                           |
|              | find . -type f -exec du -b {} \; \| sort -nr \| head -10 | 找当前目录下最大的10个文件（按字节排序）                                           |
| **SSH 远程连接** | ssh-keygen -t rsa -b 4096 -C "your_email"                | 客户端生成密钥对（一路回车）                                                  |
|              | ssh-copy-id user@服务器IP                                   | 将公钥（id_rsa.pub）内容，追加到服务器的 ~/.ssh/authorized_keys 中 # 或使用命令一键复制： |
|              | systemctl restart sshd                                   | 配置文件 `/etc/ssh/sshd_config`                                     |

---

### 二、 核心复合命令（日常高频组合）

复合命令的核心在于**管道符 `|`**（将前命令的输出作为后命令的输入）和**重定向 `>` `>>`**（输出到文件）。

#### 1. 查找并过滤（`grep`）

- **场景**：在日志中找错误，或过滤进程。
    
- **命令**：
    
      
    
    |   |
    |---|
    |`# 在 system.log 中查找包含` `"ERROR"` `的行（忽略大小写）`<br><br>`cat /var/log/system.log \| grep -i` `"error"`<br><br>`# 查看所有 Python 进程（排除 grep 自身）`<br><br>`ps aux \| grep python \| grep -v grep`<br><br>`# 更优雅的写法（用 pgrep）`<br><br>`pgrep -la python`|
    

#### 2. 统计与排序（`wc` + `sort` + `uniq`）

- **场景**：统计日志行数、访问量排序。
    
- **命令**
    
    |   |
    |---|
    |`# 统计当前目录下有多少个 .log 文件`<br><br>`ls -la \| grep` `".log"` `\| wc -l`<br><br>`# 统计 Nginx 日志中访问量最高的` `10` `个 IP`<br><br>`cat access.log \| awk` `'{print $1}'` `\| sort \| uniq -c \| sort -nr \| head -``10`<br><br>`(拆解：awk取第一列IP → sort排序 → uniq -c统计次数 → sort -nr按数字倒序 → 取前``10``)`|
    
      
    

#### 3. 替换与切割（`sed` + `awk`）

- **场景**：批量替换配置文件内容，或提取特定列。
    
- **命令**：  
    bash
    
    # 将 config.txt 中的所有 "old" 替换为 "new"（直接修改文件需加 -i）
    sed 's/old/new/g' config.txt
    
    # 查看当前目录下所有文件的大小（按列取第5列）
    ls -lh | awk '{print $5 "\t" $9}'
    

#### 4. 快速查找文件（`find` + `xargs`）

- **场景**：批量删除或移动大量文件。
    
- **命令**：
    
    |   |
    |---|
    |`# 找到当前目录下所有 .tmp 文件并删除（比 rm -rf 更可控）`<br><br>`find . -name` `"*.tmp"` `-type f \| xargs rm -f`<br><br>`# 查找最近` `7` `天内修改过的文件`<br><br>`find /home -mtime -``7` `-type f`|
    

---

### 三、 逻辑运算符复合（`&&` `||` `;`）

用于**条件执行**，常见于脚本或一键部署。

|`&&`|**前成功**才执行后（短路与）|`mkdir test && cd test` （只有创建成功才进入）|
|`\|`|**前失败**才执行后（短路或）|`command \| echo "执行失败，请检查"`|
|`;`|顺序执行，无关成败|`make clean ; make` （不管是否clean成功，都执行make）|
|`&`|放入后台运行|`python app.py &` （终端关闭任务会中止，建议用`nohup`）|
|`\`|换行续接（美化长命令）|`docker build -t myapp . \`  <br>`--build-arg env=prod`|

**实战组合技（一键更新代码）：**

|   |
|---|
|`git pull && mvn clean` `package` `&& systemctl restart myapp \| echo` `"部署失败，请回滚"`|

  

---

### 四、 高级高效技巧（必知必会）

#### 1. 命令历史快捷操作

- `!!`：执行上一条命令（权限不足时：`sudo !!`）。
    
- `!$`：上一条命令的最后一个参数（如 `mkdir test` 后，`cd !$` 直接进入）。
    
- `Ctrl + R`：**反向搜索**历史命令（输入关键字，按`Ctrl+R`循环查找）。
    

#### 2. 文本处理三板斧（文本分析神器）

|   |
|---|
|`# 查看内存前``3``高的进程（按第``4``列排序）`<br><br>`ps aux \| sort -k4 -nr \| head -``3`<br><br>`# 统计当前目录各后缀名的文件数量`<br><br>`ls \| sed` `'s/.*\.//'` `\| sort \| uniq -c`|

#### 3. 后台保持运行（`screen` / `nohup`）

- `nohup ./long_task.sh &`：即使SSH断开，任务仍在后台跑（输出写入`nohup.out`）。
    
- `screen -S task` 开启新会话，`Ctrl+A+D` 分离，`screen -r task` 恢复。
    

---

### 五、 避坑提醒（安全意识）

1. **`rm -rf`** 前务必用 `ls` 确认路径，建议用 `find` + `xargs` 替代。
    
2. **重定向覆盖**：`>` 会清空文件，想追加请用 `>>`。
    
3. **变量空格**：`a = 1` 是错的，必须是 `a=1`（等号两边无空格）。
    
4. **引号区别**：双引号`"`内`$变量`会解析，单引号`'`内全部视为纯文本。
    

[](https://out-wiki.tsingmicro.xyz/pages/viewpage.action?pageId=97149034)