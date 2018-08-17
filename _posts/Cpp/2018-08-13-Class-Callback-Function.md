---
layout: pageshow
title: 类成员函数用作回调函数的方式
category: Cpp
description: 类内函数在使用回调时的注意点
---

## 类成员函数的this指针
类成员函数分为静态成员函数和非静态成员函数，类非静态成员函数能够访问类内所有的成员是由于隐含调用了this指针；类静态成员函数没有this指针，所以只能访问类的静态变量和静态方法。
## 类静态方法的实际调用情形
```c++
class Student{
public:
    Student(const char* pszName){
        m_strName = pszName;
    }

    void Call(){
        std::cout << m_strName << std::endl;
    }
private:
    std::string m_strName;
};

void main(){
    Student student("John");
    Student.Call();
}
```
以上代码中，当调用`student.Call`时，编译器隐式引入this指针的效果，便如重写该调用成了如下形式
```c++
Student::Call(&student)
```
## 回调函数的例子
情景：老师和学生一对一辅导，首先老师要问学生名字，学生按自己的方式回答。
1. 首先定义回答方式的回调
```c++
typedef void (*ReplyCB)();
```
2. 定义老师
```c++
class Teacher{
public:
    void HasStudent(Student* st){
        m_pStudent = st;
    }
    void SetReplyCB(ReplyCB reply){
        m_ReplyCB = reply;
    }
    void AskName(){
        m_ReplyCB();
    }
private:
    Student* m_pStudent;
    ReplyCB m_ReplyCB;
};
```
3. 错误例子
```c++
Student student("John");
Teacher teacher;
teacher.HasStudent(&student);
teacher.SetReplyCB(student.Call);
teacher.AskName();
``` 
编译报错，`student.Call`的实际函数形式包含了`this`指针，所以其形式类似于`void Call(Student* st)`不符合`ReplyCB`的回调函数形式。

***

有两种方式可以正确用作回调函数
1. 不使用成员函数，为了访问类的成员变量，可以使用友元操作符(friend)，在C++中将该函数说明为类的友元即可。 

```c++
#include <iostream>
#include <string>
class Student{
friend void Call(Student* pStudent);
public:
	Student(const char* pszName){
		m_strName = pszName;
	}

private:
	std::string m_strName;
};

void Call(Student* pStudent){
	std::cout << pStudent->m_strName << std::endl;
}
typedef void (*ReplyCB)(Student* pStudent);

class Teacher{
public:
	void HasStudent(Student* st){
		m_pStudent = st;
	}
	void SetReplyCB(ReplyCB reply){
		m_ReplyCB = reply;
	}
	void AskName(){
		m_ReplyCB(m_pStudent);
	}
private:
	Student* m_pStudent;
	ReplyCB m_ReplyCB;
};

void main(){
	Student student("John");
	Teacher teacher;
	teacher.HasStudent(&student);
	teacher.SetReplyCB(Call);
	teacher.AskName();
	
	getchar();
}
```
2. 使用静态成员,访问类私有成员
```c++
#include <iostream>
#include <string>
class Student{
friend void Call(Student* pStudent);
public:
	Student(const char* pszName){
		m_strName = pszName;
	}

static void Call(Student* pStudent){
	std::cout << pStudent->m_strName << std::endl;
}
private:
	std::string m_strName;
};

typedef void (*ReplyCB)(Student* pStudent);

class Teacher{
public:
	void HasStudent(Student* st){
		m_pStudent = st;
	}
	void SetReplyCB(ReplyCB reply){
		m_ReplyCB = reply;
	}
	void AskName(){
		m_ReplyCB(m_pStudent);
	}
private:
	Student* m_pStudent;
	ReplyCB m_ReplyCB;
};

void main(){
	Student student("John");
	Teacher teacher;
	teacher.HasStudent(&student);
	teacher.SetReplyCB(Student::Call);
	teacher.AskName();
	
	getchar();
}
```
