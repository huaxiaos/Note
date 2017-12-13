# 下载

- [官网链接](https://maven.apache.org/download.cgi)
- 下载Binary zip archive

# 设置Maven环境变量

- `$ open ~/.bash_profile`
- 在文件最后添加以下内容
	- `export M2_HOME=你的maven的本地保存路径`
	- `export PATH=$PATH:$M2_HOME/bin`
- 保存
- `$ source ~/.bash_profile` 

# 设置Java环境变量

- 几个用于定位Java的命令
	- `/usr/libexec/java_home`
	- `/usr/libexec/java_home -V` 显示所有的Java版本号及路径
- 修改bash_profile文件
	- `export JAVA_HOME=你的Java的Home路径`

# 检查是否成功

- `mvn -v`   

# 参考链接

- http://www.jianshu.com/p/ecb7a9d49590
- http://yerl.cn/blog/macos-setup-maven