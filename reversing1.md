# Modified Code
![](https://img.shields.io/badge/category-reversing-blue)

## Description
Your friends, penguins, have an enemy hidden in the darkness. You crafted a warning program with the message obscured that you were going to send to them but you noticed that the enemy has intercepted and modified the code! Can you fix the code and find the original message?

The message should be submitted wrapped with CTF{ and }

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	string code = "BLCXNKQOJHYNSOQKK";
	string hash = "dEwaDmMO79UAgocrS";
	for(int i = 0;i < 13 ;i++){
		code [i] = (char)((code[i]) % 26 + 65);
	}
	cout << code << "\n";
}
```
## Solution

First of all, one should notice that the length of the strings `code` and `hash` are both length `17` and not 13. Thus, one should edit the length of the for loop to run until `17`.

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	string code = "BLCXNKQOJHYNSOQKK";
	string hash = "dEwaDmMO79UAgocrS";
	for(int i = 0;i < 17 ;i++){
		code [i] = (char)((code[i]) % 26 + 65);
	}
	cout << code << "\n";
}
```

Next, one should notice that the `hash` string is given but not used. Upon trial and error, one could find that subtracting the hash from code would obtain the most legitimate answer.

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	string code = "BLCXNKQOJHYNSOQKK";
	string hash = "dEwaDmMO79UAgocrS";
	for(int i = 0;i < 17 ;i++){
		code [i] = (char)((code[i] - hash[i]) % 26 + 65);
	}
	cout << code << "\n";
}
```

Finally, one should see the code printed out is very close to a word that can be made out. However, there are some letters that seem off. Upon inspection, one can see that the issue lies in modding. One must pad the `code[i] - hash[i]` with more `26` in order to put them all into a positive domain. Upon trial and error, one can find that padding it twice would obtain the correct answer. 

```cpp
#include<bits/stdc++.h>
using namespace std;

int main(){
	string code = "BLCXNKQOJHYNSOQKK";
	string hash = "dEwaDmMO79UAgocrS";
	for(int i = 0;i < 17 ;i++){
		code [i] = (char)((code[i] - hash[i] + 26 + 26) % 26 + 65);
	}
	cout << code << "\n";
}
```
