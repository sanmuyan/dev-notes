# Shell

## 运算符

- -eq  相等
- -ne  不相等
- -gt  大于
- -lt  小于
- -ge  大于等于
- -le  小于等于

```bash
if [[ $a -eq $b && $a -gt $c ]]; then
   echo "a等于b大于c"
fi
if [[ $a -eq $b || $a -gt $c ]]; then
   echo "a等于b或者大于c"
fi
```

## 数组

```bash
data=(a b c)
echo "数组的第一个元素 ${data[0]}"
echo "数组的所有元素 ${data[*]}"
echo "数组的长度 ${#data[*]}"

for i in ${data[*]}
do
  echo $i
done

for (( i=0;i<${#data[*]};i++ ))
do
  echo "${data[i]}"
done
```

## 流程控制

```bash
# 循环100次
for (( i=0;i<100;i++ ))
do
  echo $i
done

# 无限循环
while true
do
  command
  continue  # 跳出本次循环
  break     # 跳出循环
done

action=stop
case $action in
  stop)
    command
    ;;
  start)
    command
  *)
    command  
    ;;
esac

```

## 交互输入

```bash
s="生产环境"
# 高亮输出
echo -e "你正在操作 \033[1;33m${s}\033[0m \n\n"
# 可以回退
stty erase ^H
while true
do
  read -p  "Input env name [prod] : " n
  case $n in
   prod)
     env_name=prod
     break
   ;;
   *)
     echo "Invalid input"
   ;;
  esac
done
```

## 输入输出

```bash
command  >/dev/null 2>&1  # 全丢弃
command  2>&1 >/dev/null  # 丢弃标准输出，保留标准错误
command  1>/dev/null      # 丢弃标准输出，保留标准错误
command  2>/dev/null      # 保留标准输出，丢弃标准错误
```

## 函数

```shell

# 定义函数，可以省略function 关键字
function get_name(){
  echo $1
}

x=$(get_name 123)

echo $x
```