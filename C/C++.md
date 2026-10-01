# Visio Studio

创建项目时的解决方案指的是：解决方案（Solution）是一个容器，用于组织一个或多个相关的项目（Project）。

解决方案的主要作用是：

1. 统一管理多个项目，特别是当这些项目彼此依赖时。
2. 设置项目之间的依赖关系，确保编译顺序。
3. 管理整个解决方案的配置（如Debug、Release等）。



Ctrl + F5 快捷运行，不调试直接运行，（点击空心箭头）

**调试自动关闭控制台：**按照VS的提示可以在程序运行完成后自动关闭控制台。可以再代码中使用`system("pause")`和`cin.get()`语句防止一闪而过。

**设置为启动项目：**若一个解决方案下有多个项目，VS必须选择其中一个作为启动项目：右击项目，选择设置为启动项目



## 可执行文件

Visio Studio界面上面**本地Windows调试器**旁边可以选择**Debug(调试)**和**x64(平台)**

编译-->链接-->生成

1. 项目右击清理：清除调试过程中生成的缓存文件（在项目的x64/Debug文件目录下），如`*.obj`文件
2. 项目中的源文件处右击，选择编译，可以生成`*.obj`文件（在项目的x64/Debug文件目录下）。编译源文件，未链接
3. 解决方案右击生成，可以生成`*.exe`文件（在解决方案的x64/Debug文件目录下）





# Code

```c++
#include <iostream>

int main(){
	std::cout << "Hello World"<<std::endl;
}
```

- `#include <iostream>`：预处理，引入头文件
- `int main()`：主函数，默认`return 0`
- `<<`：输出运算符
- `std::cout`：iostream提供的标准输出流对象
- `std`：命名空间
- `::`：作用域运算符：`命名空间::对象`

命名空间是C++中用于**避免名称冲突**的一种机制，它可以将全局作用域划分为不同的命名区域，让相同名称在不同命名空间中可以共存。

```c++
#include <iostream>
using namespace std;

int main(){
	cout << "Hello World" << endl;
	//system("pause");
    cin.get();
}
```

- `system("pause")`：可以让调试控制台不要直接退出，不要一闪而过
- `cin.get()`：同样可以实现上述效果，函数以回车为结束符



## 数据基础

### 输入输出

- cin： 输入
- cout ：输出

```c++
int main(){
	cout << "Hello World" << endl;
	cout << "请输入你的名字"<<endl;
	string name;
	cin >> name;
	cout<<"你好 "<<name;
    
	//system("pause");
	cin.get();
	cin.get();
}
```

可以指定输出的进制类型，之后再使用`cout`进行输出的时候会把整数转为该进制

```c++
cout<<dec;//十进制
cout<<hex;//十六进制
cout<<oct;//八进制
cout<<bin;//二进制
//例如：
int number = 10;
cout << hex;
cout << number << endl; 
// 输出 a
```





### 函数

`返回值 函数名(参数列表){ }`

可以将函数写在其他文件中，然后在源文件进行函数声明，就可以在源文件中使用函数

### 变量

`类型 变量名`

注意要初始化，养成好习惯。

### 作用域

- 全局作用域：全局变量，默认初始化为0
- 块作用域：局部变量，必须手动初始化

```c++
int number;
int main() {
	int number = 10;
	cout << number << endl;
	cout << ::number<<endl;
}
```

`::number`：默认命名空间下，全局的number

### 常量const

**符号常量**：宏定义，本质是文本的查找替换：`#define PI 3.14`。（c++不推荐）

**使用const限定符：**`const float Pi=3.14;`，必须初始化，不可改变，常量首字母常常大写。（有变量类型，编译器可以进行安全检查）

### 类型

| 证书类型       | 注释                  |
| -------------- | --------------------- |
| bool           | 布尔，赋值非0皆为true |
| char           | 字符  8位             |
| short          | 短整型  16位          |
| int            | 整型  32位            |
| long           | 长整形  32位          |
| long long      | 长整型  64位          |
| **无符号整形** |                       |
| unsigned short | 16                    |
| unsigned ...   |                       |
| **字符类型**   |                       |
| char           | 字符                  |
| wchat_t        | 宽字符                |
| chat16_t       | Unicode字符           |
| char32_t       | Unicode字符           |
| **浮点类型**   |                       |
| float          | 32位                  |
| double         | 64位                  |
| long double    | 12/16字节             |

给变量赋值超过表达范围不会报错，但是会出现意料之外的效果。因为计算机底层只负责按照类型解释二进制，不负责检查。**涉及到字面量类型向变量类型转换。**

### 字面量

**整型字面量：**默认int类型

- 13：十进制
- 015：八进制
- 0xd：十六进制

如果超过int表达范围，选择能够表达这个数的，长度最小的那个类型。对于十进制，就选择int-->long-->long long ；对于八进制和十六进制int-->unsigned int-->long-->unsigned long-->long long-->unsigned long long

字面量加后缀：

- 默认什么都不加：默认int类型
- l和L：表示long
- ll和LL：表示long long
- u和U：表示无符号，可与上述类型组合使用



**浮点字面量：**默认double
后缀：

- f和F：表示float
- l和L：表示 long double

**字符和字符串字面量：**

字符串本质是字符数组

### 类型转换

- 赋值超过整型范围：保留低位字节，舍弃高位字节
- 整型转浮点型超过范围：丢失精度

#### 运算中的类型转换

**隐式转换：**默认向高类型转换。

**显式转换**：

```c++
//C语言风格
(double)total/count;
//C++风格
double(total)/count;
//C++强制类型转换运算符（比较靠近底层）
static_cast<double>(total)/count
```



### 位运算符

- ~ ：按位求反
- <<：左移
- \>>：右移
- &：按位与
- ^：按位异或
- |：按位或

**对于移位：**较小的整型会自动提升为int后，在进行移位，得到的结果也是int型。
左移补零，右移看符号。一般只对无符号数移位。**只能移位32位以内**





## 流程控制

#### 条件分支

```c++
int main() {
	int age = 17;
	if (age > 18) {
		cout << "Welcom! "<<age<<endl;
	}
	else if(age==18){
		cout << "Welcom! " << age << endl;
	}
	else {
		cout << "Not Welcom!";
	}
    
    // switch(表达式)。case 的值必须是整型
	switch (age) {
	case 17:
		cout << age << endl;
		break;
	case 18:
		cout << age << endl;
		break;
	default:
		cout << age << endl;
        break;
	}
}
```

条件分支可嵌套

#### while循环

```c++
int main() {
	int age = 1;
	while (age++ < 18) {
		cout << "age = " << age << endl;
	}
    
    do {
		cout << "age = " << age << endl;
	} while (age-- > 1);
}
```

#### for循环 for each

```c++
for (int age = 1; age < 18; ++age) {
    cout << "age = " << age << endl;
}

for (int num : {1, 2, 3, 4, 5, 6, 7}) {
    cout << "num = " << num << endl;
}
```

所有类型的循环都可嵌套

#### 跳转

- break：跳出当前流程控制语句
- continue：跳过当前流程控制语句，进入下一块或下一次循环
- return：
- goto：最好不要用



## 复合数据类型

### 数组

**定义：**`类型 数组名[长度]`，长度必须是常量，不能是变量

**初始化：**

- 方法一：`int arr[3] = {1,2,3}`
- 方法二：`float arr[] = {1.1, 2.1}`
- 注：若初始值数量少于“长度”，则后续位置初始化值为0。不可初始值数量多于“长度”

**访问：**`arr[0]`，访问下标为0的元素

**数组长度：**

```c++
int arr[8] = { 1,2,3,4,5,6,7,8 };
cout << "arr所占空间大小" << sizeof(arr) << endl;
cout << "每个元素所占空间大小" << sizeof(arr[0]) << endl;
int length = sizeof(arr) / sizeof(arr[0]);
cout << "数组长度" << length;
cin.get();
```



### 多维数组

**定义：**`int arr[3][4]`

**初始化：**

- 方式一：`int arr[2][3] = {{1,2,3},{4,5,6}}`，推荐方式
- 方式二：`int arr[2][3] = {1,2,3,4,5,6}`
- 注：`int arr[2][3] = {1,2,3,4}`按行初始化，其他位置0；
- 方式三：`int arr[][3] = {1,2,3,4,5,6}`，可省略第一维度。若初始化值没有填满，则后续位置初始0。

**访问：**`arr[行号][列号]`

**数组长度：**

```c++
// 数组大小
// 每行大小
// 元素大小
// 行数(数组大小 / 每行大小)
// 列数(每行大小 / 元素大小)
```

**遍历：**

1. 双层for循环

2. for each
   ```c++
   	int arr1[2][3] = { 1,2,3,4,5,6 };
   	for (auto& row : arr1) {
   		for (auto num : row) {
   			cout << " " << num;
   		}
   		//cout << endl;
   	}
   ```

   

### vector容器

`vector` 是C++标准模板库（STL）中最重要、最常用的容器之一，它提供**动态数组**功能，可以自动管理内存，支持快速随机访问。

```c++
#include <vector>
#include <iostream>

int main() {
    // 1. 空vector
    std::vector<int> v1;
    
    // 2. 指定初始大小
    std::vector<int> v2(5);          // 5个元素，都初始化为0
    std::vector<int> v3(5, 10);      // 5个元素，都初始化为10
    
    // 3. 列表初始化（C++11）
    std::vector<int> v4 = {1, 2, 3, 4, 5};
    std::vector<int> v5{10, 20, 30};  // 等价写法
    
    // 4. 从其他容器复制
    std::vector<int> v6(v4.begin(), v4.end());  // 复制v4
    std::vector<int> v7(v4);                    // 复制构造
    
    // 5. 从数组复制
    int arr[] = {1, 3, 5, 7, 9};
    std::vector<int> v8(arr, arr + 5);  // 复制数组
    
    return 0;
}
```

**遍历：**

```c++
for (int item : v7) {
    cout << item << " " << endl;
}

for (int item = 0; item < v5.size(); item++) {
    cout << v5[item] << " " << endl;
}
// 1. 正向迭代器
for(auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
}
// 4. C++11范围for循环（推荐）
for(const auto& elem : vec) {
    std::cout << elem << " ";
}
```

**添加元素：**

```c++
// 末尾添加元素（最常用）
vec.push_back(10);   // vec: [10]
vec.push_back(20);   // vec: [10, 20]
vec.push_back(30);   // vec: [10, 20, 30]

// 指定位置插入
auto it = vec.begin() + 1;  // 指向第二个元素
vec.insert(it, 15);        // vec: [10, 15, 20, 30]

// 一次插入多个元素
vec.insert(vec.end(), {40, 50});  // vec: [10, 15, 20, 30, 40, 50]

// 注意：insert在中间插入效率较低（需要移动后面所有元素）
```

**删除元素：**

```c++
std::vector<int> vec = {10, 20, 30, 40, 50, 60, 70};

// 1. 删除末尾元素
vec.pop_back();           // 删除70

// 2. 删除指定位置元素
vec.erase(vec.begin() + 2);  // 删除第三个元素（30）

// 3. 删除范围元素
vec.erase(vec.begin() + 1, vec.begin() + 3);  // 删除[20, 40)

// 4. 清空所有元素
vec.clear();              // vec变为空

// 5. 删除特定值的所有元素（搭配算法）
std::vector<int> nums = {1, 2, 3, 2, 4, 2, 5};
nums.erase(std::remove(nums.begin(), nums.end(), 2), nums.end());
// 删除所有2，nums变为[1, 3, 4, 5]
```

**二维矩阵：**

```c++
// 创建3x4的矩阵，所有元素初始化为0
std::vector<std::vector<int>> matrix(3, std::vector<int>(4, 0));

// 初始化二维数组
std::vector<std::vector<int>> matrix2 = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// 访问元素
matrix[1][2] = 10;  // 第二行第三列

// 遍历二维vector
for(int i = 0; i < matrix2.size(); ++i) {
    for(int j = 0; j < matrix2[i].size(); ++j) {
        std::cout << matrix2[i][j] << " ";
    }
    std::cout << std::endl;
}

// 范围for遍历
for(const auto& row : matrix2) {
    for(int elem : row) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;
}
```



### 字符串

引入头文件`#include<string>`。（`<iostream>`中已有引入）

**初始化：**

- 默认初始化：`string s1`
- 拷贝初始化：
  - `string s2=s1; `
  - `stirng s3="hello"`
- 直接初始化：
  - `string s4("hello")`
  - `string s5(8, 'h')`：8个‘h’字符

**访问：**与数组相同，不可越界

**长度：**`str.size()`

**字符串拼接：**用“+”可以进行拼接（运算符重载）。**两个字面量不可直接拼接，运算符两侧至少要有一个string类型**

**字符串比较：**按字符逐个比较ASCII码

**输入输出：**

- 输入：

  - 使用cin读取单词：`cin>>str1>>str2`，这种方式只能读取一个单词，遇到空格结束读取

  - getline()读取一行信息:`getline(cin, str3)`
  - cin.get()读取一个字符：`char ch = cin.get()`
    传入参数`charstr[20]; cin.get(str, 20);`

- 输出：`cout<<str;`

### 读写文件

引入头文件:`#include <fstream>`

#### 1. **ifstream - 读取文件**

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream inputFile;  // 创建输入文件流对象
    
    // 打开文件
    inputFile.open("data.txt");
    // std::ifstream inputFile("data.txt")
    
    // 检查文件是否成功打开
    if (!inputFile.is_open()) {
        std::cerr << "无法打开文件！" << std::endl;
        return 1;
    }
    
    // 读取文件内容
    // 逐行读取  最常见
    std::string line;
    while (std::getline(inputFile, line)) {
        std::cout << line << std::endl;
    }
    // 逐个单词读取
    string word;
    while (inputFile >> word)
        cout<<word<<" "<<endl;
    // 逐个字符读取
    char ch;
    wihle(inputFile.get(ch))
        cout<<ch<<" "<<endl;
    
    // 关闭文件
    inputFile.close();
    
    return 0;
}
```



#### 2. **ofstream - 写入文件**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ofstream outputFile;
    
    // 打开文件（默认覆盖模式）
    outputFile.open("output.txt");
    
    // 检查是否成功打开
    if (!outputFile) {
        std::cerr << "创建文件失败！" << std::endl;
        return 1;
    }
    
    // 写入内容
    outputFile << "Hello, World!" << std::endl;
    outputFile << "这是第二行" << std::endl;
    
    // 关闭文件
    outputFile.close();
    
    std::cout << "文件写入成功！" << std::endl;
    return 0;
}
```



#### 3. **fstream - 读写文件**

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::fstream file;
    
    // 以读写模式打开文件
    file.open("data.txt", std::ios::in | std::ios::out);
    
    if (!file.is_open()) {
        std::cerr << "打开文件失败！" << std::endl;
        return 1;
    }
    
    // 读取内容
    std::string content;
    std::getline(file, content);
    std::cout << "读取到: " << content << std::endl;
    
    // 写入新内容
    file << "\n追加的内容" << std::endl;
    
    file.close();
    return 0;
}
```



### 结构体

C++中结构体与类几乎相同，只有一点区别：

```c++
// 结构体：默认访问权限为public
struct MyStruct {
    int x;        // 默认为public
private:
    int y;        // 显式指定为private
};

// 类：默认访问权限为private
class MyClass {
    int x;        // 默认为private
public:
    int y;        // 显式指定为public
};

// 继承时的默认访问权限也不同
struct DerivedStruct : Base {  // 默认为public继承
    // ...
};

class DerivedClass : Base {    // 默认为private继承
    // ...
};
```





#### 1. **基本定义语法**

```c++
// 定义一个结构体类型
struct Person {
    // 成员变量
    std::string name;     // 姓名
    int age;              // 年龄
    double height;        // 身高（米）
    double weight;       // 体重（公斤）
    
    // 成员函数（C++中结构体可以包含函数）
    void printInfo() {
        std::cout << "姓名: " << name 
                  << ", 年龄: " << age 
                  << ", 身高: " << height << "米"
                  << ", 体重: " << weight << "公斤" << std::endl;
    }
    
    // 计算BMI指数
    double calculateBMI() {
        return weight / (height * height);
    }
};
```

#### 2. **不同定义方式**

```c++
// 方式1：先定义类型，后声明变量
struct Student {
    int id;
    std::string name;
};

Student s1;  // 声明变量

// 方式2：定义时直接声明变量（不推荐，C风格）
struct Point {
    int x;
    int y;
} p1, p2;  // 同时声明p1, p2

// 方式3：使用typedef（C风格，C++中较少用）
typedef struct {
    int hour;
    int minute;
    int second;
} Time;

Time t1;  // 声明变量

// 方式4：匿名结构体（C++11）
struct {
    std::string model;
    int year;
} car1, car2;  // car1和car2是不同的变量实例
```

#### 1. **创建和初始化**

```cpp
// 定义结构体
struct Book {
    std::string title;
    std::string author;
    double price;
    int pages;
};

int main() {
    // 方法1：依次赋值
    Book book1;
    book1.title = "C++ Primer";
    book1.author = "Stanley B. Lippman";
    book1.price = 99.9;
    book1.pages = 976;
    
    // 方法2：创建时初始化（C++11统一初始化）
    Book book2 = {"Effective C++", "Scott Meyers", 59.9, 320};
    Book book3{"The C++ Programming Language", "Bjarne Stroustrup", 129.9, 1376};
    
    // 方法3：部分初始化（剩余成员默认初始化）
    Book book4 = {"Design Patterns"};  // 只初始化第一个成员
    // author="", price=0.0, pages=0
    
    // 方法4：使用构造函数（见后文）
    
    return 0;
}
```



#### 2. **访问结构体成员**

```cpp
struct Rectangle {
    double width;
    double height;
    
    double area() {
        return width * height;
    }
};

int main() {
    Rectangle rect;
    
    // 使用点运算符访问成员
    rect.width = 10.5;
    rect.height = 20.3;
    
    std::cout << "面积: " << rect.area() << std::endl;
    
    // 结构体指针使用箭头运算符->
    Rectangle* ptr = &rect;
    ptr->width = 15.0;  // 等价于 (*ptr).width = 15.0;
    
    return 0;
}
```



#### 3. **结构体数组**

```cpp
struct Employee {
    int id;
    std::string name;
    std::string department;
    double salary;
};

int main() {
    // 创建结构体数组
    Employee employees[3] = {
        {1001, "张三", "技术部", 15000.0},
        {1002, "李四", "销售部", 12000.0},
        {1003, "王五", "市场部", 13000.0}
    };
    
    // 遍历数组
    for(int i = 0; i < 3; i++) {
        std::cout << "ID: " << employees[i].id 
                  << ", 姓名: " << employees[i].name
                  << ", 部门: " << employees[i].department
                  << ", 工资: " << employees[i].salary << std::endl;
    }
    
    // 使用范围for循环（C++11）
    for(const auto& emp : employees) {
        std::cout << emp.name << " - " << emp.department << std::endl;
    }
    
    return 0;
}
```

### 枚举

枚举是C++中一种用户定义的数据类型，它允许为**整数值赋予有意义的名称**，使代码更清晰、更易维护。

#### 基本语法

```c++
// 定义枚举类型
enum Color {
    RED,    // 默认值为0
    GREEN,  // 默认值为1
    BLUE    // 默认值为2
};

// 使用枚举
Color favoriteColor = GREEN;

// C++11引入了两种新的枚举类型
enum class ScopedColor { RED, GREEN, BLUE };  // 有作用域枚举
enum struct AltColor { RED, GREEN, BLUE };    // 与enum class相同
```

#### 有作用域枚举

```cpp
#include <iostream>

// 有作用域枚举（推荐使用）
enum class Color {
    RED,
    GREEN,
    BLUE
};

enum class TrafficLight {
    RED,    // 不会与Color::RED冲突
    YELLOW,
    GREEN
};

// 指定底层类型
enum class Priority : unsigned char {
    LOW = 1,
    MEDIUM = 5,
    HIGH = 10
};

int main() {
    Color c = Color::RED;
    TrafficLight light = TrafficLight::GREEN;
    
    // 不能隐式转换为int
    // int colorValue = c;  // 错误！
    int colorValue = static_cast<int>(c);  // 必须显式转换
    
    // 不能直接比较不同枚举类型
    // if (c == light) { }  // 错误！
    
    // 使用作用域访问
    if (c == Color::RED) {
        std::cout << "颜色是红色" << std::endl;
    }
    
    return 0;
}
```

### 指针

指针是一个**变量**，其值是**另一个变量的内存地址**。

```cpp

int num = 42;        // 普通整型变量
int* ptr = &num;     // ptr是指向num的指针

// & 地址运算符：获取变量的内存地址
// * 解引用运算符：访问指针指向的值

std::cout << "num的值: " << num << std::endl;           // 42
std::cout << "num的地址: " << &num << std::endl;        // 0x7ffeedf123
std::cout << "ptr存储的地址: " << ptr << std::endl;      // 0x7ffeedf123
std::cout << "ptr指向的值: " << *ptr << std::endl;       // 42
```

#### 基本操作

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 20;
    
    // 声明指针
    int* p;          // 指向int的指针（未初始化，危险！）
    int* p1 = nullptr; // 初始化为空指针（C++11推荐）
    
    // 指向变量
    p = &a;          // p指向a
    std::cout << "p指向的值: " << *p << std::endl;  // 10
    
    // 通过指针修改变量
    *p = 100;        // 相当于 a = 100
    std::cout << "a的新值: " << a << std::endl;     // 100
    
    // 改变指针的指向
    p = &b;          // 现在p指向b
    *p = 200;        // 相当于 b = 200
    
    std::cout << "a = " << a << ", b = " << b << std::endl;  // 100, 200
    
    // 指针的指针
    int** pp = &p;   // pp是指向指针p的指针
    std::cout << "pp指向的值: " << **pp << std::endl;  // 200
    
    return 0;
}
```

#### 指针的类型

```cpp
#include <iostream>

int main() {
    // 基本类型指针
    int* intPtr;
    double* doublePtr;
    char* charPtr;
    
    // 1 指向数组的指针
    int arr[5] = {1, 2, 3, 4, 5};
    int* arrPtr = arr;  // 数组名本身就是首元素地址
    
    // 2 指向结构体的指针
    struct Point {
        int x, y;
    };
    
    Point point = {10, 20};
    Point* pointPtr = &point;
    std::cout << "点坐标: (" << pointPtr->x << ", " << pointPtr->y << ")" << std::endl;
    
    // 3 指向指针的指针（多级指针）
    int value = 42;
    int* p1 = &value;
    int** p2 = &p1;     // 二级指针
    int*** p3 = &p2;    // 三级指针
    
    std::cout << "value = " << ***p3 << std::endl;  // 42
    
    // 4 void指针（泛型指针）
    void* voidPtr = &value;
    // *voidPtr = 50;  // 错误！void指针不能直接解引用
    int* intPtr2 = static_cast<int*>(voidPtr);  // 需要类型转换
    
    return 0;
}

```

#### const与指针

```c++
int main() {
    int a = 10;
    int b = 20;
    
    // 1. 指向常量的指针（指针本身可以修改，但不能通过它修改值）
    const int* p1 = &a;
    // *p1 = 30;  // 错误！不能修改
    p1 = &b;      // 正确！可以改变指向
    
    // 2. 常量指针（指针本身不能修改，但可以通过它修改值）
    int* const p2 = &a;
    *p2 = 30;     // 正确！可以修改值
    // p2 = &b;   // 错误！不能改变指向
    
    // 3. 指向常量的常量指针（都不能修改）
    const int* const p3 = &a;
    // *p3 = 40;  // 错误！
    // p3 = &b;   // 错误！
    
    // 记忆技巧：const在*左边修饰值，在*右边修饰指针
    
    return 0;
}
```

#### 指针运算

##### 1 指针算术运算

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr = arr;  // 指向第一个元素
    
    std::cout << "指针算术运算:" << std::endl;
    std::cout << "ptr    指向: " << *ptr << std::endl;       // 10
    std::cout << "ptr+1  指向: " << *(ptr + 1) << std::endl; // 20
    std::cout << "ptr+2  指向: " << *(ptr + 2) << std::endl; // 30
    
    // 指针递增
    ptr++;          // 现在指向arr[1]
    std::cout << "递增后: " << *ptr << std::endl;           // 20
    
    // 指针递减
    ptr--;          // 回到arr[0]
    
    // 指针比较
    int* p1 = &arr[0];
    int* p2 = &arr[3];
    
    if(p1 < p2) {
        std::cout << "p1在p2之前" << std::endl;
    }
    
    // 指针相减（得到元素个数差）
    std::cout << "p2 - p1 = " << (p2 - p1) << std::endl;  // 3
    
    // 遍历数组
    std::cout << "数组元素: ";
    for(int* p = arr; p < arr + 5; p++) {
        std::cout << *p << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

##### 2 指针与数组的关系

```cpp
#include <iostream>

int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    
    // 数组名就是指向第一个元素的指针
    std::cout << "arr    = " << arr << std::endl;
    std::cout << "&arr[0]= " << &arr[0] << std::endl;  // 与arr相同
    
    // 访问元素的等价方式
    std::cout << "arr[2]    = " << arr[2] << std::endl;      // 3
    std::cout << "*(arr+2)  = " << *(arr + 2) << std::endl;  // 3
    std::cout << "2[arr]    = " << 2[arr] << std::endl;      // 3（少见）
    
    // 多维数组指针
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    
    // 指向一维数组的指针
    int (*rowPtr)[4] = matrix;  // 指向包含4个int的数组
    
    std::cout << "matrix[1][2] = " << matrix[1][2] << std::endl;      // 7
    std::cout << "*(*(matrix+1)+2) = " << *(*(matrix+1)+2) << std::endl; // 7
    
    return 0;
}
```

```cpp
//指针数组和数组指针
// 数组指针
int (*rowPtr)[4] = matrix;  // 一个指向包含4个int的数组的指针
//指针数组
int* pa[5];//包含5个int指针的数组
```





#### 指针与函数

##### 1 **指针作为函数参数**

```cpp

#include <iostream>

// 1. 传值（创建副本）
void incrementByValue(int x) {
    x++;  // 只修改副本
}

// 2. 传指针（可以修改原变量）
void incrementByPointer(int* x) {
    if(x != nullptr) {  // 安全检查
        (*x)++;         // 修改原变量
    }
}

// 3. 传引用（C++推荐方式）
void incrementByReference(int& x) {
    x++;  // 修改原变量
}

// 4. 返回指针的函数
int* createArray(int size) {
    if(size <= 0) return nullptr;
    
    int* arr = new int[size];
    for(int i = 0; i < size; i++) {
        arr[i] = i * 10;
    }
    return arr;  // 返回动态分配的数组指针
}

// 5. 函数指针
bool compare(int a, int b) {
    return a < b;
}

void sortArray(int* arr, int size, bool (*comp)(int, int)) {
    // 使用函数指针comp进行比较
    for(int i = 0; i < size - 1; i++) {
        for(int j = 0; j < size - i - 1; j++) {
            if(comp(arr[j+1], arr[j])) {  // 使用函数指针
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

int main() {
    int a = 10;
    
    incrementByValue(a);
    std::cout << "传值后: " << a << std::endl;      // 10（未改变）
    
    incrementByPointer(&a);
    std::cout << "传指针后: " << a << std::endl;    // 11
    
    incrementByReference(a);
    std::cout << "传引用后: " << a << std::endl;    // 12
    
    // 使用返回指针的函数
    int* arr = createArray(5);
    if(arr != nullptr) {
        for(int i = 0; i < 5; i++) {
            std::cout << arr[i] << " ";
        }
        std::cout << std::endl;
        delete[] arr;
    }
    
    // 使用函数指针
    int nums[] = {5, 2, 8, 1, 9};
    sortArray(nums, 5, compare);
    
    std::cout << "排序后: ";
    for(int i = 0; i < 5; i++) {
        std::cout << nums[i] << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

##### 2. **智能指针（现代C++推荐）**

```cpp
#include <memory>  // 智能指针头文件
#include <iostream>

class Resource {
public:
    Resource() { std::cout << "Resource创建" << std::endl; }
    ~Resource() { std::cout << "Resource销毁" << std::endl; }
    void use() { std::cout << "使用Resource" << std::endl; }
};

int main() {
    // 1. unique_ptr：独占所有权（C++11）
    {
        std::unique_ptr<Resource> up1(new Resource());
        // std::unique_ptr<Resource> up2 = up1;  // 错误！不能复制
        std::unique_ptr<Resource> up2 = std::move(up1);  // 可以移动
        
        if(up1 == nullptr) {
            std::cout << "up1已被移动，为空" << std::endl;
        }
        
        up2->use();  // 使用箭头运算符访问
    }  // up2离开作用域，自动释放Resource
    
    // 2. shared_ptr：共享所有权（引用计数）
    {
        std::shared_ptr<Resource> sp1 = std::make_shared<Resource>();
        std::shared_ptr<Resource> sp2 = sp1;  // 可以复制，引用计数+1
        
        std::cout << "引用计数: " << sp1.use_count() << std::endl;  // 2
        
        {
            std::shared_ptr<Resource> sp3 = sp1;
            std::cout << "引用计数: " << sp1.use_count() << std::endl;  // 3
        }  // sp3离开作用域，引用计数-1
        
        std::cout << "引用计数: " << sp1.use_count() << std::endl;  // 2
    }  // sp1和sp2离开作用域，引用计数为0，释放Resource
    
    // 3. weak_ptr：弱引用（不增加引用计数）
    {
        std::shared_ptr<Resource> sp = std::make_shared<Resource>();
        std::weak_ptr<Resource> wp = sp;  // 弱引用
        
        std::cout << "引用计数: " << sp.use_count() << std::endl;  // 1
        
        // 使用weak_ptr前需要转换为shared_ptr
        if(auto locked = wp.lock()) {  // 尝试获取shared_ptr
            locked->use();
            std::cout << "引用计数: " << sp.use_count() << std::endl;  // 2
        }  // locked离开作用域，引用计数-1
        
        sp.reset();  // 释放Resource
        std::cout << "引用计数: " << sp.use_count() << std::endl;  // 0
        
        if(wp.expired()) {
            std::cout << "资源已释放" << std::endl;
        }
    }
    
    return 0;
}

```

### 引用

引用是C++中为已存在变量创建的**别名**，它提供了更安全、更直观的变量别名机制。

引用是变量的**别名**，它**必须初始化**，且一旦绑定到某个变量，就不能再绑定到其他变量。它并不像指针一样被分配内存，**引用没有占用内存**

```cpp

int num = 42;

// 创建引用（别名）
int& ref = num;  // ref是num的引用（别名）

// 使用引用
ref = 100;  // 实际上修改了num的值

std::cout << "num = " << num << std::endl;     // 100
std::cout << "ref = " << ref << std::endl;     // 100
std::cout << "&num = " << &num << std::endl;   // 地址相同
std::cout << "&ref = " << &ref << std::endl;   // 地址相同

// 引用不是新变量，是已存在变量的别名
```

#### 常量引用

```cpp
#include <iostream>

int main() {
    int x = 10;
    const int y = 20;
    
    // 常量引用：不能通过引用修改值
    const int& cref1 = x;     // ✅ 可以绑定到非常量
    const int& cref2 = y;     // ✅ 可以绑定到常量
    const int& cref3 = 30;    // ✅ 可以绑定到临时对象
    
    // 4. 不同类型转换
    double d = 3.14;
    const int& cref5 = d;  // ✅ 创建临时int，绑定到临时对象
    
    // 5. 函数参数中的const引用
    auto process = [](const std::string& str) {
        std::cout << "长度: " << str.length() << std::endl;
        // 不能修改str
    };
    
    process("Hello World");
    
    // cref1 = 40;            // ❌ 错误：不能通过const引用修改
    
    // const引用延长临时对象生命周期
    const std::string& str = "Hello World";  // 临时字符串生命周期被延长
    std::cout << str << std::endl;           // 安全使用
    
    // 函数参数常用const引用
    auto print = [](const std::string& msg) {
        std::cout << msg << std::endl;
        // msg[0] = 'h';      // ❌ 错误：const引用不能修改
    };
    
    print("Hello");
    
    return 0;
}

```

#### 引用作为函数参数

```cpp
#include <iostream>
#include <string>

// 1. 传值（创建副本，效率低）
void incrementByValue(int x) {
    x++;  // 只修改副本
}

// 2. 传指针（需要检查nullptr）
void incrementByPointer(int* x) {
    if(x != nullptr) {
        (*x)++;
    }
}

// 3. 传引用（推荐：效率高且安全）
void incrementByReference(int& x) {
    x++;  // 直接修改原变量
}

// 4. 传const引用（只读访问，效率高）
void printLargeObject(const std::string& str) {
    std::cout << str << std::endl;
    // str[0] = 'A';  // ❌ 错误：不能修改
}

// 5. 返回引用（注意不要返回局部变量的引用）
int& getElement(int arr[], int index) {
    return arr[index];  // ✅ 可以：返回数组元素的引用
}

int& badReturn() {
    int local = 10;
    return local;       // ❌ 危险：返回局部变量的引用
}

// 6. 链式调用（返回*this的引用）
class Counter {
private:
    int count;
public:
    Counter() : count(0) {}
    
    // 返回引用支持链式调用
    Counter& increment() {
        count++;
        return *this;  // 返回自身的引用
    }
    
    Counter& add(int value) {
        count += value;
        return *this;
    }
    
    int get() const { return count; }
};

int main() {
    int a = 10;
    
    incrementByValue(a);
    std::cout << "传值后: " << a << std::endl;      // 10（未改变）
    
    incrementByPointer(&a);
    std::cout << "传指针后: " << a << std::endl;    // 11
    
    incrementByReference(a);
    std::cout << "传引用后: " << a << std::endl;    // 12
    
    // 链式调用示例
    Counter c;
    c.increment().add(5).increment();
    std::cout << "计数器值: " << c.get() << std::endl;  // 7
    
    return 0;
}
```



## 函数

### 函数定义

```cpp
// 函数定义语法：
// 返回类型 函数名(参数列表) { 函数体 }

// 示例：加法函数
int add(int a, int b) {
    int sum = a + b;
    return sum;
}

// 无返回值函数
void printMessage() {
    std::cout << "Hello, World!" << std::endl;
}

// 无参数函数
int getRandomNumber() {
    return rand() % 100;
}

```

### 参数传递方式

```cpp
#include <iostream>

// 1. 传值（创建副本）
void modifyValue(int x) {
    x = 100;  // 只修改副本
}

// 2. 传引用（操作原变量）
void modifyReference(int& x) {
    x = 100;  // 修改原变量
}

// 3. 传指针（可以修改原变量）
void modifyPointer(int* x) {
    if(x != nullptr) {
        *x = 100;  // 修改原变量
    }
}

// 4. 传const引用（只读，高效）
void printLargeObject(const std::string& str) {
    std::cout << str << std::endl;
    // str[0] = 'A';  // 错误：不能修改
}

int main() {
    int a = 10;
    
    modifyValue(a);
    std::cout << "传值后: " << a << std::endl;      // 10（未变）
    
    modifyReference(a);
    std::cout << "传引用后: " << a << std::endl;    // 100（改变）
    
    int b = 20;
    modifyPointer(&b);
    std::cout << "传指针后: " << b << std::endl;    // 100（改变）
    
    return 0;
}

```

### 默认参数与函数重载

```cpp

// 默认参数必须在最后
void printInfo(std::string name, int age = 20, bool isStudent = true) {
    std::cout << "姓名: " << name 
              << ", 年龄: " << age
              << ", 学生: " << (isStudent ? "是" : "否") 
              << std::endl;
}


// 函数重载：同名函数，参数不同
void print(int value) {
    std::cout << "整数: " << value << std::endl;
}

void print(double value) {
    std::cout << "浮点数: " << value << std::endl;
}

void print(const std::string& value) {
    std::cout << "字符串: " << value << std::endl;
}

void print(int a, int b) {
    std::cout << "两个整数: " << a << ", " << b << std::endl;
}

// 不能仅通过返回类型重载
// int getValue() { return 1; }
// double getValue() { return 1.0; }  // 错误！

int main() {
    print(10);           // 调用print(int)
    print(3.14);         // 调用print(double)
    print("Hello");      // 调用print(const string&)
    print(10, 20);       // 调用print(int, int)
    
    return 0;
}
```





### 返回类型

```cpp
#include <iostream>
#include <vector>
#include <tuple>

// 1. 返回基本类型
int add(int a, int b) {
    return a + b;
}

// 2. 返回引用（注意生命周期）
int& getElement(std::vector<int>& vec, int index) {
    if(index >= 0 && index < vec.size()) {
        return vec[index];  // 返回引用，可以修改原元素
    }
    static int error = -1;
    return error;
}

// 3. 返回指针（通常用于动态分配）
int* createArray(int size) {
    int* arr = new int[size];
    for(int i = 0; i < size; i++) {
        arr[i] = i * 10;
    }
    return arr;
}

// 4. 返回结构体/对象
struct Point {
    int x, y;
};

Point createPoint(int x, int y) {
    return {x, y};  // C++11起可以直接返回列表
}

// 5. 返回多个值（使用pair或tuple）
std::pair<int, double> calculate(int a, int b) {
    int sum = a + b;
    double avg = (a + b) / 2.0;
    return {sum, avg};  // C++11列表初始化
}

// 6. 无返回值
void logMessage(const std::string& msg) {
    std::cout << "[LOG] " << msg << std::endl;
    // 不需要return语句
}

int main() {
    // 使用返回引用的函数修改vector
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    getElement(numbers, 2) = 100;  // 修改第三个元素
    std::cout << "numbers[2] = " << numbers[2] << std::endl;  // 100
    
    // 使用返回多个值的函数
    auto [sum, avg] = calculate(10, 20);
    std::cout << "和: " << sum << ", 平均: " << avg << std::endl;
    
    return 0;
}

```



### 函数与数组

#### 函数传递数组

##### 1. **传递原生数组（退化为指针）**

C++ 中，**当把原生数组（如 `int arr[10]`）作为参数传递时，它会自动“退化”为指向首元素的指针**。

示例：

```cpp
#include <iostream>

void printArray(int arr[], int size) {
    for (int i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;
}

int main() {
    int a[] = {1, 2, 3, 4, 5};
    printArray(a, 5);  // 数组名 a 自动转为指针
    return 0;
}
```

> ✅ **注意**：
>
> - 函数参数中 `int arr[]` 和 `int* arr` 是等价的。
> - **必须额外传入数组长度**，因为函数内部无法通过 `sizeof(arr)` 获取原始数组大小（此时 `arr` 是指针）。

------

##### 2. **使用引用传递固定大小的数组（保留大小信息）**

如果你知道数组的大小，并希望函数能“感知”这个大小，可以使用**数组引用**。

示例：

```cpp
#include <iostream>

void printArray(int (&arr)[5]) {  // 引用一个大小为5的int数组
    for (int x : arr) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
}

int main() {
    int a[5] = {10, 20, 30, 40, 50};
    printArray(a);  // 只能传大小为5的int数组
    return 0;
}
```

> ✅ **优点**：函数知道数组的确切大小。
>  ❌ **缺点**：函数只能接受特定大小的数组，不够通用。

#### 函数返回数组

在 C++ 中，**函数不能直接返回一个原生数组（如 `int[10]`）**，因为数组类型是“不可复制”的，且函数返回值必须是可复制或可移动的对象。

##### ✅ 方法 1：返回 `std::array`（固定大小数组，推荐）

适用于**编译期已知大小**的数组。

```cpp
#include <iostream>
#include <array>

std::array<int, 5> createArray() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    return arr; // 支持拷贝或移动（C++11 起高效）
}

int main() {
    auto arr = createArray();
    for (int x : arr) {
        std::cout << x << " ";
    }
    // 输出: 1 2 3 4 5
}
```

> ✅ **优点**：安全、高效、支持 STL、大小固定且已知。
>  ✅ **现代 C++ 首选方式**（C++11 及以上）。

------

##### ✅ 方法 2：返回 `std::vector`（动态大小数组，最常用）

适用于**运行时确定大小**或需要动态调整的情况。

```cpp
#include <iostream>
#include <vector>

std::vector<int> createArray(int n) {
    std::vector<int> vec(n);
    for (int i = 0; i < n; ++i) {
        vec[i] = i * 2;
    }
    return vec; // 返回值优化（RVO）使其高效
}

int main() {
    auto v = createArray(4); // {0, 2, 4, 6}
    for (int x : v) {
        std::cout << x << " ";
    }
}
```

> ✅ **优点**：灵活、自动管理内存、支持任意大小。
>  ✅ **绝大多数场景下的最佳选择**。

------

##### ⚠️ 方法 3：返回指针（不推荐，除非特殊需求）

```cpp
// ❌ 不推荐！
int* createArray() {
    int* arr = new int[5]{10, 20, 30, 40, 50};
    return arr;
}

int main() {
    int* p = createArray();
    // ... 使用 p ...
    delete[] p; // 必须手动释放！容易忘记
}
```

> ❌ **问题**：
>
> - 调用者必须记得 `delete[]`；
> - 不符合 RAII 原则；
> - 异常不安全。

✅ **替代方案**：如果必须用指针语义，改用智能指针（见下文）。

##### 4 尾置返回类型

```cpp
auto fun(int x) -> int(*)[5];
```



### 内联函数

在 C++ 中，**内联函数（inline function）** 是一种编译器优化机制，用于**建议编译器将函数调用“展开”为实际的函数体代码**，而不是执行常规的函数调用（即压栈、跳转、返回等操作）。其主要目的是**减少函数调用的开销**，尤其适用于**短小、频繁调用的函数**。

```cpp
inline int add(int a, int b) {
    return a + b;
}

```

```cpp
class Math {
public:
    int multiply(int x, int y) {
        return x * y;  // 自动视为 inline
    }
};
```



普通函数在头文件中定义会导致**多重定义错误**（多个 `.cpp` 文件包含该头文件时）。但 `inline` 函数**允许多次定义**，只要所有定义完全相同，链接器会自动处理。

```cpp
#ifndef UTILS_H
#define UTILS_H

inline int square(int x) {
    return x * x;
}

#endif
```

其他文件可以安全地 `#include "utils.h"`，不会链接错误。



### 函数指针

**函数指针（function pointer）** 是一种指向**函数的指针变量**。它存储的是函数在内存中的入口地址，可以通过该指针**调用函数**，也可以将函数作为参数传递或返回，从而实现**回调、策略模式、动态绑定**等高级功能。

通用形式：

```cpp
返回类型 (*指针名)(参数列表);
```

```cpp
// 声明一个指向 "int func(int, int)" 类型函数的指针
int (*funcPtr)(int, int);
```

> ⚠️ 注意：`*funcPtr` 必须用括号括起来！
>  ❌ 错误写法：`int *funcPtr(int, int);` → 这是一个返回 `int*` 的函数声明！

#### 赋值与调用

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

int main() {
    // 声明函数指针
    int (*op)(int, int);

    // 赋值：函数名就是函数地址（可加 &，也可不加）
    op = add;          // 或 op = &add;

    // 通过指针调用函数
    std::cout << op(5, 3) << std::endl;  // 输出 8

    op = subtract;
    std::cout << op(5, 3) << std::endl;  // 输出 2

    return 0;
}
```



#### 函数指针作为参数（实现回调）

```cpp
void applyOperation(int a, int b, int (*op)(int, int)) {
    std::cout << "Result: " << op(a, b) << std::endl;
}

int multiply(int x, int y) {
    return x * y;
}

int main() {
    applyOperation(4, 5, multiply);  // 传入函数指针
    applyOperation(10, 3, [](int a, int b) { return a + b; }); // C++11 起也支持 lambda（需匹配签名）
    return 0;
}
```

#### 函数指针数组（实现分发表）

```cpp
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }

int main() {
    // 函数指针数组
    int (*ops[3])(int, int) = {add, sub, mul};

    std::cout << ops[0](6, 2) << std::endl; // 8
    std::cout << ops[1](6, 2) << std::endl; // 4
    std::cout << ops[2](6, 2) << std::endl; // 12

    return 0;
}
```

#### 使用 `typedef` 或 `using` 简化声明（推荐！）

c风格

```c
typedef int (*MathFunc)(int, int);
MathFunc ptr = add;
```

C++风格

```cpp
using MathFunc = int(*)(int, int);
MathFunc ptr = add;
```





## 头文件

### 1. **头文件的命名与扩展名**

- 传统上使用 `.h` 扩展名，如 `stdio.h`、`math.h`。
- 在 C++ 中，标准库头文件通常**没有扩展名**，如 `<iostream>`、`<vector>`。
- 用户自定义头文件一般使用 `.h` 或 `.hpp`（后者更常用于 C++）。

------

### 2. **头文件的作用**

- **声明函数和类**：告诉编译器这些函数或类存在，并描述它们的接口。
- **避免重复定义**：通过**头文件保护**（include guards 或 `#pragma once`）防止多次包含导致的重复定义错误。
- **提高编译效率和模块化**：将接口（头文件）与实现（源文件 `.cpp`）分离，便于代码组织和重用。

------

### 3. **示例**

#### 头文件 `math_utils.h`

```cpp
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// 函数声明
int add(int a, int b);

#endif // MATH_UTILS_H
```

或者使用 `#pragma once`（更简洁，但非标准，不过主流编译器都支持）：

```cpp
#pragma once

int add(int a, int b);
```

#### 源文件 `math_utils.cpp`

```cpp
#include "math_utils.h"

// 函数定义
int add(int a, int b) {
    return a + b;
}
```

#### 主程序 `main.cpp`

```cpp
#include <iostream>
#include "math_utils.h"  // 包含自定义头文件

int main() {
    std::cout << add(3, 4) << std::endl;  // 输出 7
    return 0;
}
```

------

### 4. **包含方式**

- **系统/标准库头文件**：用尖括号 `<>`，如 `#include <iostream>`
- **用户自定义头文件**：用双引号 `""`，如 `#include "myheader.h"`

------

### 5. **注意事项**

- 头文件中**通常只放声明**，不放定义（除非是内联函数、模板等特殊情况）。
- 避免在头文件中定义非 `const` 全局变量，否则可能引发多重定义错误。
- 使用**头文件保护**防止重复包含。