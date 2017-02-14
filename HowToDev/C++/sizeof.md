# Sizeof的研究
## 非类大小计算
### sizeof计算的原则如下：
1. 数据成员对齐规则：
    - 结构(struct)(或联合(union))的数据成员，第一个数据成员放在offset为0的地方，
    - 以后每个数据成员存储的起始位置要从该成员大小或者成员的子成员大小;
    - (只要该成员有子成员，比如说是数组，结构体等)的整数倍开始(比如int在32位机为４字节,则要从4的整数倍地址开始存储。
2. 结构体作为成员:
    - 如果一个结构里有某些结构体成员,则结构体成员要从其内部最大元素大小的整数倍地址开始存储;
    - (struct a里存有struct b,b里有char,int ,double等元素,那b应该从8的整数倍开始存储)

3. 收尾工作:
    - 结构体的总大小,也就是sizeof的结果,.必须是其内部最大成员的整数倍,不足的要补齐;

```
typedef struct bb {
 int id;             //[0]....[3]
 double weight;      //[8].....[15]　　　　　　原则1
 float height;       //[16]..[19],总长要为8的整数倍,补齐[20]...[23]　　　　　原则3
} BB;

typedef struct aa {
 char name[2];      //[0],[1]
 int  id;           //[4]...[7]　　　　　　　　　　原则1

 double score;      //[8]....[15]　　　　
 short grade;       //[16],[17]　　　　　　　　
 BB b;              //[24]......[47]　　　　　　　　　　原则2
} AA;

int main() {
  AA a;
  cout<<sizeof(a)<<" "<<sizeof(BB)<<endl;
  return 0;
}
```
结果：
```
48 24
```
### 再讲讲#pragma pack()
在代码前加一句#pragma pack(1),你会很高兴的发现,上面的代码输出为
```
32 16
```

说明：
- bb是4+8+4=16,aa是2+4+8+2+16=32;

这不是理想中的没有内存对齐的世界吗.没错,#pragma pack(1),告诉编译器,所有的对齐都按照1的整数倍对齐,换句话说就是没有对齐规则。

## 类大小计算
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