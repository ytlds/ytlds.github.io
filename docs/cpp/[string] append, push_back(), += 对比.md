## 结论：



push back：用于单字符

+=：单字符、字符数组、C 字符串



append：不能用单字符

![image.png](https://cdn.nlark.com/yuque/0/2020/png/2402403/1598701692866-4247cbbc-0a1e-4ea6-b533-3fbb6d4c8561.png)





转载：https://www.geeksforgeeks.org/stdstringappend-vs-stdstringpush_back-vs-operator-c/

std::string::append vs std::string::push_back() vs Operator += in C++



- Operator **+= :** appends single-argument values.
- **append() :** lets you specify the appended value by using multiple arguments.
- **[push_back() : ](https://www.geeksforgeeks.org/stdstringpush_back-in-cpp/)**lets you to append single character at a time.

## 1)Full String:

- **+= :** We can append full string using +=.
- **append() :** We can also append full string using append().
- **push_back :** doesn’t allow appending of full string.

```c++
// CPP code for comparison on the 
// basis of appending Full String 
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
void appendDemo(string str1, string str2) 
{ 
    string str = str1; 
  
    // Appending using += 
    str1 += str2; 
    cout << "Using += : "; 
    cout << str1 << endl; 
  
    // Appending using append() 
    str.append(str2); 
    cout << "Using append() : "; 
    cout << str << endl; 
} 
  
// Driver code 
int main() 
{ 
    string str1("Hello World! "); 
    string str2("GeeksforGeeks"); 
  
    cout << "Original String : " << str1 << endl; 
    appendDemo(str1, str2); 
  
    return 0; 
} 
```



Output:

```
Original String : Hello World! 
Using += : Hello World! GeeksforGeeks
Using append() : Hello World! GeeksforGeeks
```



## 2)Appending part of String:

- **+= :** Doesn’t allow appending part of string.
- **append() :** Allows appending part of string.
- **push_back :** We can’t append part of string using push_back.*
  *

```c++
// CPP code for comparison on the basis of 
// Appending part of string 
  
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
void appendDemo(string str1, string str2) 
{ 
    // Appends 5 characters from 0th index of 
    // str2 to str1 
    str1.append(str2, 0, 5); 
    cout << "Using append() : "; 
    cout << str1; 
} 
  
// Driver code 
int main() 
{ 
    string str1("GeeksforGeeks "); 
    string str2("Hello World! "); 
  
    cout << "Original String : " << str1 << endl; 
    appendDemo(str1, str2); 
  
    return 0; 
} 
```

Output:

```
Original String : GeeksforGeeks 
Using append() : GeeksforGeeks Hello
```



## 3)Appending C-string (char*):

- **+= :** Allows appending C-string
- **append() :** It also allows appending C-string
- **push_back :** We cannot append C-string using push_back().

```c++
// CPP code for comparison on the basis of 
// Appending C-string 
  
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
void appendDemo(string str) 
{ 
    string str1 = str; 
  
    // Appending using += 
    str += "GeeksforGeeks"; 
    cout << "Using += : "; 
    cout << str << endl; 
  
    // Appending using append() 
    str1.append("GeeksforGeeks"); 
    cout << "Using append() : "; 
    cout << str1 << endl; 
} 
  
// Driver code 
int main() 
{ 
    string str("World of "); 
  
    cout << "Original String : " << str << endl; 
    appendDemo(str); 
  
    return 0; 
} 
```

Output:

```
Original String : World of 
Using += : World of GeeksforGeeks
Using append() : World of GeeksforGeeks
```



## 4)Appending character array:

- **+= :** Allows appending of character array
- **append() :** Allows appending of character array.
- **push_back :** Does not allow char array appending.

```c++
// CPP code for comparison on the basis of 
// Appending character array 
  
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
void appendDemo(string str) 
{ 
    char ch[6] = {'G', 'e', 'e', 'k', 's', '\0'};  
    string str1 = str; 
  
    // Appending using += 
    str += ch; 
    cout << "Using += : " << str << endl; 
  
    // Appending using append() 
    str1.append(ch); 
    cout << "Using append() : "; 
    cout << str1 << endl; 
} 
  
// Driver code 
int main() 
{ 
    string str("World of "); 
  
    cout << "Original String : " << str << endl; 
    appendDemo(str); 
  
    return 0; 
} 
```

Output:

```
Original String : World of 
Using += : World of Geeks
Using append() : World of Geeks
```



## 5)Appending single character:

- **+= :** We can append single character using += operator.
- **append() :** Doesn’t allow appending single character.
- **push_back :** Allows appending single character.

```c++
// CPP code for comparison on the basis of 
// Appending single character 
  
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
void appendDemo(string str) 
{ 
    string str1 = str; 
  
    // Appending using += 
    str += 'C'; cout << "Using += : " << str << endl; 
  
    // Appending using push_back() 
    str1.push_back('C'); 
    cout << "Using push_back : "; 
    cout << str1; 
} 
  
// Driver code 
int main() 
{ 
    string str("AB"); 
  
    cout << "Original String : " << str << endl; 
    appendDemo(str); 
  
    return 0; 
} 
```

Output:

```
Original String : AB
Using += : ABC
Using push_back : ABC
```



6)**Iterator range:**

- **+= :** Doesn’t provide iterator range.
- **append() :** Provides iterator range.
- **push_back :** Doesn’t provide iterator range.

```c++
// CPP code for comparison on the basis of 
// Appending using iterator range 
  
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
void appendDemo(string str1, string str2) 
{ 
  
    // Appends all characters from 
    // str2.begin()+5, str2.end() to str1 
    str1.append(str2.begin() + 5, str2.end()); 
    cout << "Using append : "; 
    cout << str1; 
} 
// Driver code 
int main() 
{ 
    string str1("Hello World! "); 
    string str2("GeeksforGeeks"); 
  
    cout << "Original String : " << str1 << endl; 
    appendDemo(str1, str2); 
  
    return 0; 
} 
```

Output:

```
Original String : Hello World! 
Using append : Hello World! forGeeks
```



7)**Return Value:**

- **+= :** Return *this.
- **append() :** Returns *this
- **push_back :** Doesn’t return anything.

```c++
// CPP code for comparison on the basis of 
// Return value 
  
#include <iostream> 
#include <string> 
using namespace std; 
  
// Function to demonstrate comparison among 
// +=, append(), push_back() 
string appendDemo(string str1, string str2) 
{ 
    // Appends str2 in str1 
    str1.append(str2); // Similarly with str1 += str2 
    cout << "Using append : "; 
  
    // Returns *this 
    return str1; 
} 
  
// Driver code 
int main() 
{ 
    string str1("Hello World! "); 
    string str2("GeeksforGeeks"); 
    string str; 
    cout << "Original String : " << str1 << endl; 
    str = appendDemo(str1, str2); 
    cout << str; 
    return 0; 
} 
```

Output:

```
Original String : Hello World! 
Using append : Hello World! GeeksforGeeks
```