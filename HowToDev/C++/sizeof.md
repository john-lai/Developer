# Sizeof的研究

## 类大小
### 空类的占用空间
```
class Base {  
public:  
    Base();  
    ~Base();  
  
};
```

- 注意到我这里显示声明了构造跟析构，但是sizeof(Base)的结果是1。

- 因为一个空类也要实例化，所谓类的实例化就是在内存中分配一块地址，每个实例在内存中都有独一无二的地址。

- 同样空类也会被实例化，所以编译器会给空类隐含的添加一个字节，这样空类实例化之后就有了独一无二的地址了。所以空类的sizeof为1。

- 析构函数，跟构造函数这些成员函数，是跟sizeof无关的，也不难理解因为我们的sizeof是针对实；

- 而普通成员函数，是针对类体的，一个类的成员函数，多个实例也共用相同的函数指针，所以自然不能归为实例的大小。

### 非空类的占用空间
```
class Base {  
public:  
    Base();                  
    virtual ~Base();         //每个实例都有虚函数表  
    void set_num(int num)    //普通成员函数，为各实例公有，不归入sizeof统计  
    {  
        a=num;  
    }  
private:  
    int  a;                  //占4字节  
    char *p;                 //4字节指针  
};  
  
class Derive:public Base {  
public:  
    Derive():Base(){};       
    ~Derive(){};  
private:  
    static int st;         //非实例独占  
    int  d;                     //占4字节  
    char *p;                    //4字节指针  
  
};  
  
int main()   {   
    cout<<sizeof(Base)<<endl;  
    cout<<sizeof(Derive)<<endl;  
    return 0;  
}  
```
结果：
```
12
20
```
说明：
- Base类里的int  a;char *p; 占8个字节；
- 虚析构函数virtual ~Base(); 的指针占4子字节;
- 其他成员函数不归入sizeof统计;
- Derive类首先要具有Base类的部分，也就是占12字节;
- int  d;char *p; 占8字节;
- static int st; 不归入sizeof统计;

在考虑在Derive里加一个成员char c,
```
class Derive:public Base {  
public:  
    Derive():Base(){};  
    ~Derive(){};  
private:  
    static int st;  
    int  d;  
    char *p;  
    char c;  
  
}; 
```
结果：
```
12
24
```
说明：
- 一个char c; 增加了4字节，说明类的大小也遵守类似class字节对齐的补齐规则；

### 至此，我们可以归纳以下几个原则：
1. 类的大小为类的非静态成员数据的类型大小之和，也就是说静态成员数据不作考虑；
2. 普通成员函数与sizeof无关；
3. 虚函数由于要维护在虚函数表，所以要占据一个指针大小，也就是4字节；
4. 类的总大小也遵守类似class字节对齐的，调整规则； 