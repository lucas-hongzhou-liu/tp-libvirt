# Shell 脚本入门知识点（Bash）

适合在 WSL / Linux 下边看边练。建议保存后用编辑器打开，配合终端实操。

---

## 1. 什么是 Shell 脚本

Shell 是命令行解释器；脚本就是把一串命令按逻辑写进文件，再一次性执行。

常见 shell：`bash`（最常用）、`sh`、`zsh`。

在 WSL 里默认一般是 bash，可用下面命令确认：

```bash
echo $SHELL
bash --version
```

---

## 2. 第一个脚本

```bash
#!/bin/bash
# 上面这一行叫 shebang，告诉系统用 bash 执行本文件

echo "Hello, WSL"
```

保存为 `hello.sh`，然后：

```bash
chmod +x hello.sh   # 赋予可执行权限
./hello.sh          # 执行
# 或者不改权限，直接：
bash hello.sh
```

---

## 3. 变量

```bash
#!/bin/bash

name="Alice"        # 赋值：等号两边不要空格
echo "Hello, $name"
echo "Hello, ${name}"   # 推荐：花括号更清晰，尤其变量名旁边还有别的字符时

# 只读变量
readonly PI=3.14

# 删除变量
unset name

# 命令替换：把命令输出赋给变量
today=$(date +%F)
echo "今天是 $today"
```

注意：

- `"$name"`：有引号时，空格和特殊字符更安全
- `$name` 未加引号时，可能被拆词（word splitting）

---

## 4. 特殊参数（重点记忆）

| 符号 | 含义 |
|------|------|
| `$0` | 脚本自己的名字（路径） |
| `$1` `$2` … | 第 1、第 2… 个位置参数 |
| `$#` | 位置参数的个数 |
| `"$@"` | 所有位置参数（保留各自边界，推荐） |
| `"$*"` | 所有位置参数拼成一个字符串 |
| `$?` | 上一条命令的退出码（0=成功，非 0=失败） |
| `$$` | 当前 shell 的 PID |
| `$!` | 最近一个后台进程的 PID |

示例：

```bash
#!/bin/bash
# 用法: ./demo.sh alice bob

echo "脚本名: $0"
echo "参数个数: $#"
echo "第一个参数: $1"
echo "第二个参数: $2"
echo "所有参数: $@"

ls /tmp >/dev/null
echo "ls 的退出码: $?"

sleep 3 &
echo "后台 sleep 的 PID: $!"
echo "当前脚本 PID: $$"
```

判断上一条命令是否成功：

```bash
grep "pattern" file.txt
if [ $? -eq 0 ]; then
  echo "找到了"
else
  echo "没找到"
fi

# 更常见写法（直接用 if 判断命令）：
if grep -q "pattern" file.txt; then
  echo "找到了"
fi
```

---

## 5. 条件判断

### 5.1 `test` / `[ ]` / `[[ ]]`

```bash
#!/bin/bash

a=10
b=20

# 数字比较：-eq -ne -lt -le -gt -ge
if [ "$a" -lt "$b" ]; then
  echo "$a 小于 $b"
fi

# 字符串
name="tom"
if [ -z "$name" ]; then
  echo "空字符串"
elif [ "$name" = "tom" ]; then
  echo "是 tom"
else
  echo "其他"
fi

# 文件相关
file="/etc/passwd"
if [ -f "$file" ]; then echo "普通文件存在"; fi
if [ -d "/tmp" ]; then echo "目录存在"; fi
if [ -r "$file" ]; then echo "可读"; fi
if [ -w "$file" ]; then echo "可写"; fi
if [ -x "/bin/ls" ]; then echo "可执行"; fi
```

常用文件测试：

| 选项 | 含义 |
|------|------|
| `-f` | 普通文件且存在 |
| `-d` | 目录且存在 |
| `-e` | 存在（任意类型） |
| `-r` / `-w` / `-x` | 可读 / 可写 / 可执行 |
| `-s` | 文件存在且非空 |
| `-z str` | 字符串长度为 0 |
| `-n str` | 字符串长度非 0 |

`[[ ]]` 是 bash 增强写法，支持 `=~` 正则、`&&` `||` 等，推荐在 bash 脚本里用：

```bash
if [[ "$name" == t* ]]; then
  echo "以 t 开头"
fi
```

---

## 6. 分支与循环

### if / elif / else

```bash
#!/bin/bash
score=85

if [ "$score" -ge 90 ]; then
  echo "优秀"
elif [ "$score" -ge 60 ]; then
  echo "及格"
else
  echo "不及格"
fi
```

### case

```bash
#!/bin/bash
os="$1"

case "$os" in
  linux|Linux)
    echo "Linux 系统"
    ;;
  windows|Windows)
    echo "Windows / WSL 场景常见"
    ;;
  *)
    echo "未知: $os"
    ;;
esac
```

### for

```bash
#!/bin/bash

for i in 1 2 3; do
  echo "i=$i"
done

for i in {1..5}; do
  echo "编号 $i"
done

for f in /tmp/*.log; do
  echo "处理文件: $f"
done

# C 风格
for ((i=0; i<3; i++)); do
  echo "C-style i=$i"
done
```

### while

```bash
#!/bin/bash
n=1
while [ "$n" -le 3 ]; do
  echo "n=$n"
  n=$((n + 1))
done
```

### 读文件每一行

```bash
#!/bin/bash
while IFS= read -r line; do
  echo "行内容: $line"
done < /etc/hosts
```

---

## 7. 算术运算

```bash
#!/bin/bash
a=3
b=5

echo $((a + b))
echo $((a * b))
echo $((a ** 2))   # 幂

# 自增
((a++))
echo "$a"
```

---

## 8. 函数

```bash
#!/bin/bash

say_hello() {
  local who="$1"   # local：函数内局部变量
  echo "Hello, $who"
}

say_hello "World"

add() {
  echo $(($1 + $2))
}

sum=$(add 3 4)
echo "3+4=$sum"
```

函数里的 `$1` `$2` 是**传给函数**的参数，不是脚本参数。

---

## 9. 输入输出与重定向

```bash
echo "普通输出"
echo "错误信息" >&2          # 输出到 stderr

echo "写入" >  out.txt       # 覆盖写入
echo "追加" >> out.txt       # 追加
cmd < input.txt              # 从文件读入标准输入
cmd >out.txt 2>&1            # stdout+stderr 都进 out.txt
cmd >/dev/null 2>&1          # 全部丢弃
```

管道：

```bash
ps aux | grep bash | wc -l
```

---

## 10. 数组（bash）

```bash
#!/bin/bash
fruits=("apple" "banana" "cherry")

echo "${fruits[0]}"      # 第一个
echo "${fruits[@]}"      # 全部
echo "${#fruits[@]}"     # 长度

fruits+=("date")         # 追加
for f in "${fruits[@]}"; do
  echo "$f"
done
```

---

## 11. 实用小脚本示例

### 11.1 检查命令是否存在

```bash
#!/bin/bash
if command -v curl >/dev/null 2>&1; then
  echo "curl 已安装"
else
  echo "请先安装 curl"
  exit 1
fi
```

### 11.2 带参数校验的脚本

```bash
#!/bin/bash
# 用法: ./backup.sh <源目录> <目标目录>

if [ "$#" -ne 2 ]; then
  echo "用法: $0 <源目录> <目标目录>"
  exit 1
fi

src="$1"
dst="$2"

if [ ! -d "$src" ]; then
  echo "源目录不存在: $src"
  exit 1
fi

mkdir -p "$dst"
cp -a "$src"/. "$dst"/
echo "备份完成: $src -> $dst"
```

### 11.3 简单菜单

```bash
#!/bin/bash
echo "1) 看日期"
echo "2) 看谁在线"
echo "3) 退出"
read -r -p "请选择: " choice

case "$choice" in
  1) date ;;
  2) who ;;
  3) exit 0 ;;
  *) echo "无效选项" ;;
esac
```

### 11.4 出错即停（调试友好）

```bash
#!/bin/bash
set -euo pipefail
# -e : 任一命令失败就退出
# -u : 使用未定义变量时报错
# -o pipefail : 管道中任一失败都算失败

echo "安全模式已开启"
```

调试时还可以：

```bash
bash -x script.sh    # 打印每一条实际执行的命令
```

---

## 12. 引号与常见坑

```bash
name="Alice Bob"

# 错误：会被拆成两个词
# touch $name

# 正确：
touch "$name"

# 单引号：原样，不展开变量
echo '$name'    # 输出字面量 $name

# 双引号：展开变量
echo "$name"    # 输出 Alice Bob
```

路径、用户输入、`$1`、命令替换结果，**能加双引号就加**。

遍历参数请用：

```bash
for arg in "$@"; do
  echo "arg=$arg"
done
```

不要用未加引号的 `$*` / `$@`。

---

## 13. 退出码约定

| 退出码 | 含义（约定） |
|--------|----------------|
| `0` | 成功 |
| `1` | 一般错误 |
| `2` | 用法错误（如参数不对） |
| `127` | 命令未找到 |

脚本里：

```bash
exit 0   # 成功结束
exit 1   # 失败结束
```

---

## 14. WSL 下练习建议

1. 在 Windows 的 WSL 终端进入 Linux 家目录：`cd ~`
2. 建目录：`mkdir -p ~/learn-shell && cd ~/learn-shell`
3. 用 `nano hello.sh` 或 `vim hello.sh` 写脚本
4. `chmod +x hello.sh && ./hello.sh`
5. 对照本文第 4 节，专门练一遍 `$0 $1 $# "$@" $?`

Windows 与 WSL 路径互通示例：

```bash
# 访问 Windows C 盘
ls /mnt/c/Users/

# 在 WSL 里编辑，文件也可放到 /mnt/c/... 方便资源管理器打开
```

注意：脚本若放在 `/mnt/c/...`，有时会遇到可执行权限或换行符（CRLF）问题。  
若出现 `$'\r': command not found`，说明是 Windows 换行，可转换：

```bash
sed -i 's/\r$//' script.sh
# 或
dos2unix script.sh
```

学习阶段更推荐把脚本放在 WSL 的 Linux 文件系统里（如 `~/learn-shell`）。

---

## 15. 速查清单

```text
变量:          name="value"; echo "$name"
位置参数:      $0 $1 $2 ... $# "$@"
退出码:        $?
PID:           $$   后台 PID: $!
判断:          [ ]  或  [[ ]]
数字比较:      -eq -ne -lt -le -gt -ge
文件判断:      -f -d -e -r -w -x
循环:          for / while
分支:          if / elif / else / case
算术:          $((a + b))
函数:          name() { local x="$1"; ... }
安全模式:      set -euo pipefail
调试:          bash -x script.sh
```

---

## 16. 下一步可以练什么

1. 写一个脚本：接收文件名，判断是否存在并打印大小  
2. 写一个脚本：批量把当前目录 `.txt` 改名为 `.bak`  
3. 写一个脚本：循环检测某个进程是否在跑，不在则打印告警  
4. 把 `set -euo pipefail` 加进你的脚本，体会它如何帮你尽早发现错误  

学完这些，再去看复杂自动化（如测试框架里的 bash 包装、virt 相关脚本）会轻松很多。
