---
title: Move Semantics
date: 2020-12-06 07:51:25
tags: [Move Semantics]
categories: [C++]
---

# Rvalue and Lvalue

When entering the world of C++, we might hear a lot of people talking about rvalue and lvalue, but what exactly are they and why we care about knowing them? Understanding how everything behind the scene works will render us more flexibility and control of the tools we are using like in this case C++. In a very simplified way, Lvalue usually refers to a value that occupies a specific piece of memory that identifies the value, which is also called "locator value". Rvalue refers to a data value that is at some temporary memory address, which we can not use it to identify the value. This might seem abstract at first, but don't worry I'll give some examples which will demonstrate these two concepts pretty well:

{% codeblock lang:c++ %}
  int i = 3; // i occupies a specific memory location on stack, 3 is a data value which is a rvalue.
  1 = 3; // Error, cannot assign a data value to a rvalue. Do not make sense.
  i = 3; // Correct, change the value at the location where i resides on stack. When we access i, we are interpreting the value at i's memory location.
  i + 1 = 2; // Error, since i + 1 evaluates to rvalue, here i + 1 is an integer.
{% endcodeblock %}

# Reference

There are also two types of references: rvalue reference and lvalue reference. Reference is like adding another name to the value. Thus, we can not tag name to an object that does not exist. Also, we need to assign corresponding reference type to the value type (lvalue or rvalue).

{% codeblock lang:c++ %}
  int& a; // Error, don't make sense to tag a name to nothing
  int i = 3; // Initialize an integer on stack
  int &lref = i; // lref is a lvalue reference of i and manipulating a will reflect on i

  int &&rref{5}; // rref is a rvalue reference that holds a value 5
{% endcodeblock %}

Now, let's take a look at move semantics. Normally, in order to write efficient c++ code, we want to add rvalue reference copy constructor and copy operator= that takes a rvalue reference. Essentially, a move means that we want to move the resource of one object into another object instead of copying the resource. Rvalue indicates that the value is temporary and will be destroyed soon, thus we can safely steal the resource from it.

{% codeblock lang:c++ %}
class Entity
{
public:

  Entity(int x)
  {
    allocated_ptr = malloc(x); // We allocate some space of x bytes on heap
  }

  // Move copy constructor
  Entity(Entity&& other)
  {
    allocated_ptr = other.allocated_ptr; // we set the pointer value to the allocated space of the other entity

    other.allocated_ptr = nullptr; // we set pointer value of the other entity to nullptr
  }

  // Move copy method
  Entity& operator=(Entity&& other)
  {
    if (&other == this) return *this; // We need to check whether the passed in argument is the object itself

    allocated_ptr = other.allocated_ptr; // we set the pointer value to the allocated space of the other entity

    other.allocated_ptr = nullptr; // we set pointer value of the other entity to nullptr

    return *this;
  }

  ~Entity() {
    free(allocated_ptr); // free nullptr does not do anything
  }
private:
  void* allocated_ptr;
};

int main() {
  Entity x(5); // allocate space in Entity x
  Entity a = std::move(x); // allocated space is moved from x to a, and std::move(T) will cast T to &&T
}
{% endcodeblock %}

In the above example, we can see that in move constructor or move copy method, there is no new space allocated but only assign operations performed. This tool allows us to better manage our meomry space usage.
