#### Mac下SFTP连接服务器下载文件

> 由于工作上遇到了关于连接sftp服务器的问题，故记录下来
>
> 2018-08-08 HZ

- 连接服务器

  `sftp /*username@/*host`

- 系统会提示你

   ```shell
  The authenticity of host 'host (xx.xxx.xx.xx)' can't be established.
  ECDSA key fingerprint is SHA256:xxxxxxxxx/xxxxxxxxxxxxxxxxxxxxxxxxxxx.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added 'host,xx.xxx.xx.xx' (ECDSA) to the list of known hosts.
  username@host's password：
  
   ```

- 输入密码

  ```shell
  Connected to host.
  sftp>
  ```

- pwd（获取当前目录文件夹）

- cd至文件所在目录文件夹

- 利用get命令去下载文件至当前目录下（cmd打开的目录）

  ```shell
  stfp>
  get xxx.file
  Fetching /xxx/xx/xxx.file to xxx.file
  /xxx/xxx/xxx.file                                                                                                                                        100%   90MB   9.0MB/s   00:10
  ```

  到这里你就下载成功了

- 上传目录文件

   ```shell
  stfp>
  put xxx/xxx.file
   ```

  最后贴一些linux常用命令

  ```shell
  pwd: Print working directory of remote host
  lpwd: Print working directory of local client
  cd: Change directory on the remote host
  lcd: Change directory on the local client
  ls: List director on the remote host
  lls: List directory on the local client
  mkdir: Make directory on remote host
  lmkdir: Make directory on local client
  get: Receive file from remote host to local client
  put: Send file from local client to remote host
  help: Display help text
  ```

  