# Classes II

## Special Member Function

我們已經提到了 default constructor 屬於 special member functions 中的其中一種  
在繼續下一個`class`的設計之前，希望在此先將會用到的另外幾個 special member functions 一併討論  

在 C++ 中，有六種 special member functions  
以下的程式碼片段在說明怎麼樣宣告會被認為是使用者自訂的該 special member function  
並不是該 special member function 如果被隱式地產生時，會有怎樣的型式  

*   **default constructor**  
    ```C++
    class_name();
    ```
    沒有任何 parameters，或是所有 parameters 都有 default arguments  
    
    在沒有提供任何 arguments 時呼叫的 constructor  

*   **copy constructor**  
    ```C++
    class_name( cv_qualifiers class_name & );
    ```
    第一個 parameter 型別為`cv_qualifiers class_name &`  
    並且如果有其他 parameters，必須都有 default arguments  
    
    如果有需要 copy initialization 的時候，便是呼叫此 constructor  
    ```C++
    struct Foo {};
    void Test(Foo arg) {}
    
    Foo b; // b is default-initialized
    Foo a = b; // a is copy-initialized
    Test(a); // arg is copy-initialized
    ```

*   **copy assignment operator**  
    ```C++
    return_type class_name::operator= ( cv_qualifiers class_name & );
    return_type class_name::operator= ( cv_qualifiers class_name );
    ```
    只能有一個 parameter，型別為`cv_qualifiers class_name &`或`cv_qualifiers class_name`  
    
    ```C++
    struct Foo {};
    
    Foo a; // a is default-initialized
    Foo b = a; // b is copy-initialized
    b = a; // copy assignment
    ```

*   **move constructor**  
    ```C++
    class_name( cv_qualifiers class_name && );
    ```
    第一個 parameter 型別為`cv_qualifiers class_name &&`  
    並且如果有其他 parameters，必須都有 default arguments  
    
    如同 copy constructor，但第一個 argument 必須為 rvalue  

*   **move assignment operator**  
    ```C++
    return_type class_name::operator= ( cv_qualifiers class_name && );
    ```
    只能有一個 parameter，型別為`cv_qualifiers class_name &&`  
    
    如同 copy assignment operator，但 argument 必須為 rvalue  

*   **destructor**
    ```C++
    ~class_name();
    virtual ~class_name();
    ```
    
    既然有像是 constructor 那樣在物件初始化時呼叫的 function  
    那麼相對應的，destructor 是在物件即將被銷毀被呼叫的 function  

以上就是 C++ 中的 special member functions  
當一個`class`裡面沒有特地去定義這些 special member functions 時，編譯器可能會隱式地宣告  
並且在需要時，會隱式地定義這些 special member functions  

在接下來的實作中，我們可以看到這些 special member functions 是如何幫助我們進行資源管理  

## Destructor

這次我們想要實作的是`Vector`，interface 是模仿`std::vector`，但是功能被精簡許多  
簡單來說，可以把`Vector`想成是一個被封裝的 integer array  
我們能夠更簡單地插入、刪除元素，並且進行複製、查詢大小等操作  

在此，我們首先要介紹的是 destructor  
當一個物件要被銷毀時，譬如由`new`產生的物件被`delete`時  
或是因為離開 scope 導致 local variables 即將被銷毀的時候  
destructor 都會被呼叫  

利用 constructor 與 destructor 的特性，可以讓資源管理更為輕鬆  
在 C 的時候，得自己利用`malloc`與`free`來配置動態的 array 並且刪除  
但現在利用 C++ 的語法，只要將「資源的初始化」放在 constructor，「資源的釋放」放在 destructor  
就可以配合變數本身的 lifetime 對資源進行更好的管理，而不需要自己在意什麼時候要如何處理  

```C++
#include<cstddef>
#include<iostream>
#include<algorithm>

class Vector
{
public:
	Vector()
		:mBegin(nullptr), mLast(nullptr), mEnd(nullptr)
	{
		std::cout << "Default constructor" << std::endl;
	}

	std::size_t capacity() const
	{
		return mEnd - mBegin;
	}

	std::size_t size() const
	{
		return mLast - mBegin;
	}

	int& operator[](const std::size_t pos)
	{
		return mBegin[pos];
	}

	const int& operator[](const std::size_t pos) const
	{
		return mBegin[pos];
	}

	void push_back(const int val)
	{
		if (mEnd == mLast)
		{
			auto cap = std::max(capacity() + 1, capacity() * 3);
			reserve(cap);
		}

		*mLast = val;
		mLast++;
	}

	void pop_back()
	{
		if (size() > 0)
		{
			mLast--;
		}
	}

	void insert(const std::size_t pos, const int val)
	{
		if (capacity() < size() + 1)
		{
			auto cap = std::max(capacity() + capacity() / 2, capacity() + 1);
			reserve(cap);
		}

		std::copy_backward(mBegin + pos, mLast, mLast + 1);
		std::fill(mBegin + pos, mBegin + pos + 1, val);
		mLast += 1;
	}

	void erase(const std::size_t pos)
	{
		std::copy(mBegin + pos + 1, mLast, mBegin + pos);
		mLast--;
	}

	void reserve(const std::size_t new_capacity)
	{
		if (capacity() < new_capacity)
		{
			auto* start = new int[new_capacity];
			std::copy(mBegin, mLast, start);
			delete[] mBegin;

			auto s = size();
			mBegin = start;
			mLast = mBegin + s;
			mEnd = mBegin + new_capacity;
		}
	}

	void resize(const std::size_t new_size)
	{
		reserve(new_size);

		if (mLast < mBegin + new_size)
			std::fill(mLast, mBegin + new_size, 0);

		mLast = mBegin + new_size;
	}

	~Vector()
	{
	    std::cout << "Destructor" << std::endl;
	    
		delete[] mBegin;
		mBegin = mLast = mEnd = nullptr;
	}

private:
	int* mBegin;
	int* mLast;
	int* mEnd;
};
```

程式碼看上去應該會有很多未解之處  
首先我們要先對`Vector`的運作有些基礎的認知  

*   `push_back(n)`  
    在`Vector`的尾端插入值為`n`的元素  
    
*   `pop_back`  
    刪除在`Vector`尾端的元素  
    
*   `insert(pos,n)`  
    在`Vector`的第`pos`個元素前面插入值為`n`的元素  
    
*   `erase(pos)`  
    刪除在`Vector`的第`pos`個元素  

還有，對於`Vector`來說，`capacity`與`size`是不同的想法  
`size`是`Vector`當下有幾個元素，`capacity`則是代表`Vector`在配置陣列時可以存放的元素數量  
用罐子來比喻，`size`是現在罐子中溶液的存量，`capacity`則是罐子最多可以裝多少溶液  

這樣做的好處是，譬如`Vector`支援`push_back`的操作，也就是從`Vector`尾端新增一個元素  
如果`size`等同於`capacity`，就代表每次新增元素的話，都必須配置一塊新的陣列  
將舊的陣列內的元素全部複製到新的陣列以後，刪除舊的陣列以後，才能在尾端插入新元素  

隨著`Vector`的`size`漸增，插入一個元素所需花費的時間就越多，這樣的操作代價是相對高的  
所以與其這樣，不如在一開始配置陣列的時候，就比要存放的元素總數要再多一些  
這樣子在`size`超過`capacity`之前，`push_back`就會是很簡單的操作  
而在每次要重新配置陣列時，要如何決定`capacity`的大小，則是看實作如何，在此就不深入討論  

而對於使用者來說，在有些狀況可能會知道`Vector`最極端的狀況會是幾個元素  
這時候就可以先利用`reserve`來指定`capacity`，如此一來能更有效地避免重新配置陣列的操作  

`resize`則是將元素的數量擴大或是縮減，假設現在`Vector`的`size`為 5：如果是`resize(3)`  
可以想像成是呼叫了兩次`pop_back`；如果是`resize(7)`，則可以想像成呼叫了兩次`push_back(0)`  

了解了概念以後，就來看看三個 data members 各自的意義

*   `mBegin`  
    指向配置的陣列的起始位址  

*   `mLast`  
    指向的位址是`mBegin + size`  

*   `mEnd`  
    指向的位址是`mBegin + capacity`  

從上面的定義可以發現到，實際上存放著使用者插入的元素的區間是`[ mBegin, mLast )`  
而配置的陣列的有效區間則是`[ mBegin, mEnd )`  

換句話說，`mLast`是指向最後一個元素的下個元素的位址  
`mEnd`是指向配置陣列時，所能存放的最後一個元素的下一個元素的位置  

譬如

```C++
mBegin = new int[5];
mLast = mBegin;
mEnd = mBegin + 10;

mBegin[0] = 1; mLast++;
mBegin[1] = 2; mLast++;
mBegin[2] = 3; mLast++;
```

 `mBegin + 0` | `mBegin + 1` | `mBegin + 2` | `mBegin + 3` | `mBegin + 4` | `mBegin + 5`   
--------------|--------------|--------------|--------------|--------------|--------------  
100           |99            |98            |?             |?             |?               
↑`mBegin`     |              |              |↑`mLast`      |              |↑`mEnd`         

這樣的設計或許看上去很奇怪，但只要考慮到，如果`mLast`實際上是指向最後一個儲存的元素  
那麼很多時候，對於「`Vector`中沒有儲存任何元素」與「`Vector`中儲存一個元素」的處理將會相當麻煩  
讀者們如有興趣，可以在本章結束後自行改寫看看，便能理解  

了解了每個 member functions 想完成甚麼功能以後，剩下的就是實作部分  
其中可以注意到，對於`operator[]`有 const 與 non-const，這是決定`vec[0] = 100;`合不合法的關鍵  
而`size`與`capacity`也都是 const member functions  

其中，有使用到`std::fill`、`std::copy`、`std::copy_backward`這些 functions  
請自行查詢資料並試著了解如何使用，其中可以注意到，對於存取區間的有效範圍是`[ first, last )`的狀況  
在這些 functions 是屢見不鮮  

回到本節標題，destructor 就是負責把配置的陣列給刪除  
可以將`Vector`變數定義在各種地方觀察一下執行結果  

```C++
/* Vector defined as above*/

int main()
{
    Vector a;
    a.push_back(3);
    a.push_back(4);
    a.push_back(5);
    
    {
        Vector b;
        b.resize(10);
    }

    return 0;
}
```





























