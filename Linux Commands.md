# 常用文件操作

## 内容列表

```bash
ls 显示目录内容
ls -a 包含隐藏文件
ls -l 列出文件详细信息
ls -lh 显示文件大小
```

## 目录管理

```bash
cd ~trace 进入用户宿主目录
cd 进入当前家目录
cd /tmp 进入根下目录
pwd 显示当前工作目录
du -ah 显示目录大小
du -sh /etc 列出总大小
mkdir tmp 创建 tmp 目录
mkdir -p /a/b/c 递归创建目录
rm file.f 删除文件
rm -r tmp 递归删除目录
rm -rf tmp 强制删除
```

## 文件管理

```bash
cp file.f /tmp 复制文件至 tmp 目录
cp *.f /tmp 复制所有 .f 文件至 tmp 目录
mv file f 将 file 改名为 f
mv file /tmp 移动文件至 tmp 目录
head -n 2 显示前 2 行内容
tail -n 5 显示后 5 行内容
cat 显示全部
cst -n 带行号
whereis passwd 查找命令为 passwd 的文件
find / -name file.f 查找文件名为 file.f 的文件
find / -size +10000k 查找大于 10Mb 的文件
```

# 终端操作

```bash
^C 中断命令或进程
^L 清空终端屏幕
^D 退出终端
^A 移动光标至所在行首
^E 移动光标至所在行尾
^W 擦除一个单词
^U 擦除从当前光标位置到行首的全部内容
^K 擦除从当前光标位置到行尾的全部内容
^Y 粘贴快捷键擦除的内容
```
