================
Basic cheatsheet
================

.. contents:: Table of Contents
    :backlinks: none

Negative Array index
--------------------

.. code-block:: cpp

    #include <iostream>

    int main(int argc, char *argv[]) {
        // note: arr[i] = *(a + i)
        int arr[] = {1, 2, 3};
        int *ptr = &arr[1];

        std::cout << ptr[-1] << "\n";
        std::cout << ptr[0] << "\n";
        std::cout << ptr[1] << "\n";
    }


Reference
---------

.. code-block:: cpp

    #include <iostream>

    template<typename T>
    void f(T& param) noexcept {}
    // param is a reference

    int main(int argc, char *argv[])
    {
        int x = 123;
        const int cx = x;
        const int &rx = x;

        f(x);   // type(param) = int&
        f(cx);  // type(param) = const int&
        f(rx);  // type(param) = const int&

        return 0;
    }


.. code-block:: cpp

    #include <iostream>

    template<typename T>
    void f(T&& param) noexcept {}
    // param is a universal reference

    int main(int argc, char *argv[])
    {
        int x = 123;
        const int cx = x;
        const int &rx = x;

        f(x);   // x is a lvalue, type(param) = int&
        f(cx);  // cx is a lvalue, type(param) = const int&
        f(rx);  // rx is a lvalue, type(param) = const int&
        f(12);  // 12 is a rvalue, type(param) = int&&

        return 0;
    }

.. code-block:: cpp

    #include <iostream>

    template<typename T>
    void f(T param) noexcept {}
    // param is neither a pointer nor a reference.

    int main(int argc, char *argv[])
    {
        int x = 123;
        const int cx = x;
        const int &rx = x;

        f(x);   // type(param) = int
        f(cx);  // type(param) = int
        f(rx);  // type(param) = int
        f(12);  // type(param) = int

        return 0;
    }

auto
----

.. code-block:: cpp

    auto x = 123;        // type(x) = int
    const auto cx = x;   // type(cx) = const int
    const auto &rx = x;  // type(rx) = const int&

    auto &&urx = x;      // type(urx) = int&
    auto &&urcx = cx;    // type(urcx) = const int&
    auto &&urrx = rx;    // type(urrx) = const int&
    auto &&urrv = 12;    // type(urrv) = int&&

decltype(auto)
--------------

The ``decltype(auto)`` is similar to auto, which decudes type via compiler.
However, ``decltype(auto)`` preserves types reference and cv-qualifiers, while
auto does not.

.. code-block:: cpp

    #include <type_traits>

    int main(int argc, char *argv[]) {
      int x;
      const int cx = x;
      const int &crx = x;
      int &&z = 0;

      // decltype(auto) preserve cv-qualifiers
      decltype(auto) y1 = crx;
      static_assert(std::is_same<const int &, decltype(y1)>::value == 1);
      // auto does not preserve cv-qualifiers
      auto y2 = crx;
      static_assert(std::is_same<int, decltype(y2)>::value == 1);
      // decltype(auto) preserve rvalue reference
      decltype(auto) z1 = std::move(z);
      static_assert(std::is_same<int &&, decltype(z1)>::value == 1);
    }

``decltype(auto)`` is especially useful for writing a generic function's return.

.. code-block:: cpp

    #include <type_traits>

    auto foo(const int &x) {
      return x;
    }

    decltype(auto) bar(const int &x) {
      return x;
    }

    int main(int argc, char *argv[]) {
      static_assert(std::is_same<int, decltype(foo(1))>::value == 1);
      static_assert(std::is_same<const int &, decltype(bar(1))>::value == 1);
    }

Reference Collapsing
--------------------

.. code-block:: cpp

    // T& & -> T&
    // T& && -> T&
    // T&& & -> T&
    // T&& && -> T&&
    // note & always wins. that is T& && == T&& & == T& & == T&
    // only T&& && == T&&

Perfect Forwarding
------------------

.. code-block:: cpp

    #include <iostream>
    #include <utility>
    #include <type_traits>

    template <typename T>
    T&& forward(typename std::remove_reference<T>::type& t) noexcept {
      std::cout << std::is_lvalue_reference<decltype(t)>::value << std::endl;
      return static_cast<T&&>(t);
    }

    template <typename T>
    T&& forward(typename std::remove_reference<T>::type&& t) noexcept {
      static_assert(
        !std::is_lvalue_reference<T>::value,
        "Can not forward an rvalue as an lvalue."
      );
      std::cout << std::is_lvalue_reference<decltype(t)>::value << std::endl;
      return static_cast<T&&>(t);
    }

    int main (int argc, char *argv[])
    {
      int a = 0;
      forward<int>(a);     // forward lvalues to rvalues
      forward<int>(9527);  // forward rvalues to rvalues
      return 0;
    }

.. code-block:: cpp

    #include <iostream>
    #include <utility>
    #include <type_traits>

    template <typename T, typename Func>
    void wrapper(T &&a, Func fn) {
      fn(std::forward<T>(a)); // forward lvalue to lvalues or rvalues
    }

    struct Foo {
      Foo(int a1, int a2) : a(a1), b(a2), ret(0) {}
      int a, b, ret;
    };

    int main (int argc, char *argv[])
    {
      Foo foo{1, 2};
      Foo &bar = foo;
      Foo &&baz = Foo(5, 6);

      wrapper(foo, [](Foo foo) {
        foo.ret =  foo.a + foo.b;
        return foo.ret;
      });
      std::cout << foo.ret << std::endl;

      wrapper(bar, [](Foo &foo) {
        foo.ret = foo.a - foo.b;
        return foo.ret;
      });
      std::cout << bar.ret << std::endl;

      // move an rvalue to lvalue
      wrapper(std::move(baz), [](Foo &&foo) {
        foo.ret = foo.a * foo.b;
        return foo.ret;
      });
      std::cout << baz.ret << std::endl;
      return 0;
    }

Bit Manipulation
----------------

.. code-block:: cpp

	#include <iostream>
	#include <bitset>

	int main(int argc, char *argv[]) {
		std::bitset<4> b{8};

		// show number of bits set
		std::cout << b.count() << "\n";
		// compare with int
		std::cout << (b == 8) << "\n";
	}



