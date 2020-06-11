# 基础篇

## 数字-字符串转换：

使用stringstream将字符串转为数字时，有以下优点：

- 支持正负数字转换

- 不以数字开头的字符串，默认转出为0

  

但是，要注意测试空字符串场景。比如：提前判断空串

```
long toLong(string str)
{
    size_t pos = str.find_first_not_of(" ");
    if (pos == string::npos) {
        return 0;
    }

    stringstream ss;
    ss << str;
    long value;
    ss >> value;
    return value;
}
```

或者，初始化value为0，否则空字符串转换时，不会修改value值，将取系统初始化value时赋的默认值。

```
class Solution {
public:
    int myAtoi(string str) {
        long value = toLong(str);
        if (value > INT_MAX) {
            return static_cast<int>(INT_MAX);
        } else if (value < INT_MIN) {
            return static_cast<int>(INT_MIN);
        } else {
            return static_cast<int>(value);
        }
    }

    long toLong(string str)
    {
        stringstream ss;
        ss << str;
        long value = 0;
        ss >> value;
        return value;
    }
};
```

## 变量生命周期：

### 局部变量生命周期：

```
class ObjectB;

class ObjectA {
public:
    ObjectA()
    {
        std::cout << "Create ObjectA!" << std::endl;
    }

    ~ObjectA()
    {
        std::cout << "Release ObjectA!" << std::endl;
    }

public:
    shared_ptr<ObjectB> obj_;
};

class ObjectB {
public:
    ObjectB()
    {
        std::cout << "Create ObjectB!" << std::endl;
    }

    ~ObjectB()
    {
        std::cout << "Release ObjectB!" << std::endl;
    }

public:
    shared_ptr<ObjectA> obj_;
};
```

对于普通的局部变量，先声明后释放：

```
ObjectA objA;
ObjectB objB;
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectB!
Release ObjectA!

对于堆上的指针，按delete顺序先后释放：

```
ObjectA* objA(new ObjectA());
ObjectB* objB(new ObjectB());

delete objA;
delete objB;
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectA!
Release ObjectB!

对于智能指针（以shared_ptr为例，其他智能指针也这样），和堆栈局部变量一样，先声明的后释放：

```
shared_ptr<ObjectA> objA(new ObjectA());
shared_ptr<ObjectB> objB(new ObjectB());
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectB!
Release ObjectA!

shared_ptr循环指向时，只申请不释放：

```
shared_ptr<ObjectA> objA(new ObjectA());
shared_ptr<ObjectB> objB(new ObjectB());

objA->obj_ = objB;
objB->obj_ = objA;
```

回显：

Create ObjectA!
Create ObjectB!

shared_ptr如果不存在循环指向时，不存在未释放问题：

```
shared_ptr<ObjectA> objA(new ObjectA());
shared_ptr<ObjectB> objB(new ObjectB());

objA->obj_ = objB;
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectA!
Release ObjectB!

```
shared_ptr<ObjectA> objA(new ObjectA());
shared_ptr<ObjectB> objB(new ObjectB());

objB->obj_ = objA;
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectB!
Release ObjectA!

### 组合关系中，声明周期

```
class ObjectA {
public:
    ObjectA()
    {
        std::cout << "Create ObjectA!" << std::endl;
    }

    ~ObjectA()
    {
        std::cout << "Release ObjectA!" << std::endl;
    }
};

class ObjectB {
public:
    ObjectB()
    {
        std::cout << "Create ObjectB!" << std::endl;
    }

    ~ObjectB()
    {
        std::cout << "Release ObjectB!" << std::endl;
    }

public:
    ObjectA obj_;
};
```

创建时，先创建类包含的成员变量，再创建本身；释放时，先释放本身，再释放包含的成员变量。

```
ObjectB obj;
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectB!
Release ObjectA!

### 继承关系中，生命周期：

```
class ObjectA {
public:
    ObjectA()
    {
        std::cout << "Create ObjectA!" << std::endl;
    }

    ~ObjectA()
    {
        std::cout << "Release ObjectA!" << std::endl;
    }
};

class ObjectB : public ObjectA {
public:
    ObjectB()
    {
        std::cout << "Create ObjectB!" << std::endl;
    }

    ~ObjectB()
    {
        std::cout << "Release ObjectB!" << std::endl;
    }
};
```

创建时，先创建父类，在创建子类；析构时，先释放子类，再释放父类。

```
ObjectB obj;
```

回显：

Create ObjectA!
Create ObjectB!
Release ObjectB!
Release ObjectA!