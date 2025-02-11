# Lua

## 数据类型

```lua
x = "hello world"
print("字符串类型", type(x))

x = 1
print("数字类型", type(x))

x = true
print("布尔类型", type(x))

x = nil
print("空值类型", type(x))

x = {}
print("表类型", type(x))

x = function()
end
print("函数类型", type(x))

x = coroutine.create(function()
end)
print("线程类型", type(x))

x = io.stdin
print("自定义类型", type(x))
```

## 循环

```lua
for i = 1, 10 do
    print(i)
end
```

```lua
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end
```

```lua
local i = 1
while i <= 10 do
    print(i)
    i = i + 1
end
```

## 流程控制

```lua
x = 1
y = 10

if x > 0 and y == 10 then
    print("x > 0 and y == 10")
end

if x > 0 or y == 10 then
    print("x > 0 or y == 10")
elseif x == 5 then
    print("x == 5")
else
    print("x <= 0 and y != 10")
end
```

## 函数

```lua
function number_sum(a, b)
    return a + b
end

x = number_sum(1, 2)

print(x)

```

```lua
function number_sum(...)
    local sum = 0
    local arg = {...}
    for i, v in ipairs(arg) do
        sum = sum + v
    end
    return sum
end

x = number_sum(1, 2)

print(x)
```

## 数组

```lua
array = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

for i = 1, #array do
    print(array[i])
end
```

## Table

```lua
my_table = {}

my_table[1] = "Hello"
my_table[2] = "World"

for k, v in pairs(my_table) do
    print(k, v)
end

table.insert(my_table, 3, "Lua")
table.remove(my_table, 1)

for k, v in pairs(my_table) do
    print(k, v)
end

print("连接元素", table.concat(my_table, ", "))
```


## 错误处理

```lua
function number_sum(a, b)
    assert(type(a) == "number", "a is not a number")
    assert(type(b) == "number", "b is not a number")
    return a + b
end

print(number_sum(1, "x"))
```

```lua
function number_sum(a, b)
    return a + b
end

if pcall(number_sum, 1, "x") then
    print("success")
else
    print("failed")
end
```

```lua
function number_sum(a, b)
    return a + b
end

function err_handler(err)
    print(err)
end

status = xpcall(number_sum, err_handler, 1, "x")

print(status)
```

## 面向对象

```lua
-- 元类
Person = {name = ""}

function Person:new(o, name)
    o = o or {}
    setmetatable(o, self)
    self.__index = self
    self.name = name
    return o
end

-- 基类方法
function Person:print_name()
    print(self.name)
end

-- 创建对象
p = Person:new(nil, "Bob")
p:print_name()


-- 定义一个Studet类继承自Person
Student = Person:new()

function Student:new(o, name, age)
    o = o or Person:new(o, name)
    setmetatable(o, self)
    self.__index = self
    self.age = age
    return o
end

-- 重写父类方法
function Student:print_name()
    print(self.name .. " " .. self.age)
end

-- 创建对象
s = Student:new(nil, "Alice", 18)
s:print_name()

```

## I/O

```lua
-- 打开一个文件并打印全部内容

file = io.open("test.txt", "r")

print(file:read("*a"))

io.close(file)

-- 在文件最后一行添加一行内容
file = io.open("test.txt", "a")
file:write("\nHello World")

io.close(file)
```

## 协同程序

```lua
x = coroutine.create(function(s)
    for i = 1, 10 do
        if i == 2 then
            print(coroutine.status(x))
            print(coroutine.running())
            return
        end
        -- 挂起线程
        coroutine.yield()
    end
end)

coroutine.resume(x, "Hello World")
-- 此时线程已经挂起
status = coroutine.status(x)
print(status)

coroutine.resume(x, "Hello World")
-- 此时线程已经结束
status = coroutine.status(x)
print(status)
```