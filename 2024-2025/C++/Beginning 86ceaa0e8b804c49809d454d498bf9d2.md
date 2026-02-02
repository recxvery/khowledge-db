# Beginning

![image.png](2024-2025/C++/Beginning/image.png)

данная строчка позволяет на использовать всё в будущем без std.

То есть код может дальше выглядеть вот так:

c

```cpp
#include <iostream>

using namespace std;

int main(int argc, char* argv[])
{
//	int a = 42;
//	int b = 10;
	int age = 18;
	string first = "Vladimir";
	string last = "Mikhalyov";
	string name = first + ' '+ last;
	cout << name << ":" << age << '\n' << "Hello World!" << endl;
	return 0;
}
```

# Vector’а

![image.png](2024-2025/C++/Beginning/image%201.png)

## size_t

```cpp
	for (size_t i = 0; i < vec.size(); i++)
	{
		std::cout << vec[i] << "\n";
	}
```

выводим элементы вектора, size_t используется для того чтобы не переполнять памятьб

### auto

```cpp
#include <iostream>
#include <vector>

int main(int argc, char* argv[])
{
	std::vector <int> vec;
	vec.push_back(42.3f);
	vec.push_back(10.1f);
	vec.push_back(17.42f);

	for (int a : vec)
	{
		std::cout << a << "\n";
	}

	return 0;
}

```

Вот допустим мы захотели через цикл вывести числа в векторе. Однако получится так что из-за того что “a” объявлена как целое число у нас и выведутся целые элементы вектора:

![image.png](2024-2025/C++/Beginning/image%202.png)

Чтобы такого не было можем использовать auto, что будет само по себе определять тип данных: 

```cpp

int main(int argc, char * argv[])
{
	std::vector <float> vec;
	vec.push_back(42.3f);
	vec.push_back(10.1f);
	vec.push_back(17.42f);

	for (auto a : vec)
	{
		std::cout << a << "\n";
	}

	return 0;
}
```

![image.png](2024-2025/C++/Beginning/image%203.png)