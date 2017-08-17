# Classes

## 前言

在 C 裡面，我們透過 structures 來定義自己的資料型別，藉此在處理某些問題時更為簡易  
而在 C++ 裡面，更是進一步地將 structures 與 classes 的功能擴展，藉此可更直接地支援 object-oriented programming  

而促使我們需要 structures 與 classes 的理由，主要是 data abstraction 與 encapsulation  
宗旨是希望在設計一個資料型別時，能夠將 interface 與 implementation 分開，用更為高階的角度去撰寫程式  
比起只專注在如何達到目的，我們還會關注於如何讓這個物件被使用，與其他物件又該如何協作  
而在使用一個別人已經設計好的 abstract data type 時，我們就不需要去了解背後的實作，只要了解如何使用就好  

而以下介紹的僅只是 C++ 做為一個程式語言，有哪些手段可以幫助我們設計出一個 abstract data type  
並不代表掌握了這些語法，就可以寫出很漂亮的程式架構  

> Note:  
> 在 C++ 中，`class`與`struct`的差別僅止於預設的 access control 不同  
> 為方便起見，下文在介紹語法時主要都以`class`作為關鍵字，但請記住這點  

在往下看以前，請先注意幾件事情  

1.  術語部分，本文盡量採用英文，是因為有的術語採用中文，閱讀起來有時會造成障礙  

2.  有些術語可能沒看過、或是有的地方並沒有講解得很細，是因為 C++ 的語法真的不是簡單一份講義就可以說完  
    有興趣的讀者可以自行用關鍵字找資料，或是閱讀 C++ Primer 等書籍  
    也因為很多語法部分會相互牽扯，故本文也並不會針對每個主題都附上參考連結  
    然而因保留了英文術語，故找尋資料時不致沒有線索可循，關於此點請多包涵  

3.  以下的程式碼，是以筆者當下認為較好的寫法進行介紹，非常有可能有更好的設計方法可以使用  
    如有興趣，讀者也可以自行改寫，看怎麼樣的設計會更為貼切，並試著提出理由說服自己  

4.  標題一方面也是配合程式碼進行語法介紹，相關的主題可能分散到不同段落  
    所以如同第二點，當有疑問時，請上網查閱資料或是參考書籍，避免只有片段的了解  

5.  當有不確定語法如何的時候，請盡量以最保險的方式撰寫程式碼  
    譬如不知道甚麼時候變數會被預設為`0`，那就每個地方都初始化也無妨  

## Data Members, Member Functions, Access Control

假設我們想要在程式中設計一個 abstract data type 來表達有理數  
起初我們可以先利用類似 C 的`struct`設計出以下的型別  

```C++
class Rational
{
    int mNumerator;
    int mDenominator;
};

Rational a;
// a.mNumerator = 5;
// a.mDenominator = 1;
```

可以看到定義`class`的方法與 C 定義`struct`的方法十分相似  
不過現在我們不用再寫上`typedef`，就可以直接透過`class`的名稱宣告變數  
其中可以注意到，在底下有兩行程式碼被註解掉，原因是是這樣的存取操作會違反 access control  

> Note:  
> 同理，在 C++ 中，`struct`也不需要`typedef`就可以直接用其名稱  

在 C++ 中，有三個關鍵字：`public`、`protected`、`private`可以掌控其 members 的存取方式  

*   **public**  
    最為寬鬆的存取權限，可在所有地方被存取  

*   **protected**  
    僅限於該類別的 derived class 與該類別的 members 和 friend 可存取  

*   **private**  
    僅限於該類別的 members 和 friend 可存取  

而在`class`中，如果定義 members 時並沒有指定權限，那麼預設的 members 都會是`private`  
在`struct`中，則是預設為`public`  

```C++
class A
{
    int mX; // private
public:
    int mY; // public
    int mZ; // public
private:
    int mW; // private
};

struct B
{
    int mX; // public
public:
    int mY; // public
    int mZ; // public
private:
    int mW; // private
};
```

回到原本的例子，我們可以利用 access specifiers 來讓那些 data members 不隨意被外界存取  
接著，我們可以定義 member functions 來作為給使用者的 interface  

```C++
class Rational
{
public:
    int GetNumerator()
    {
        return mNumerator;
    }
    
    int GetDenominator()
    {
        return mDenominator;
    }
    
    void SetNumerator(const int num)
    {
        mNumerator = num;
    }
    
    void SetDenominator(const int den)
    {
        mDenominator = den;
    }

private:
    int mNumerator;
    int mDenominator;
};

Rational a;
a.SetNumerator(1);
a.SetDenominator(1);
```

在`class`或是`struct`中可以宣告或定義 member functions  
我們在上面的例子裡宣告並定義了四個 member functions，並且將其設定為`public`  
代表不只 class 本身，外界也可以使用這四個我們所提供的 interface  
而使用這些 member functions 的方式如同 data members 一樣，可以透過`.`或是`->`  

又，在 C++ 中，如果在 class 的定義裡面順便定義了 member functions，那麼他會被隱式地設定為 inline functions  
如果不需要 member functions 被隱式地加上`inline`，那麼建議應該將其宣告與定義分開  
這樣在`Rational`被多個程式碼檔案引用時，就不會在每個檔案編譯時都編譯一次其 member functions  

```C++
/* Rational.h */
#ifndef RATIONAL_H_
#define RATIONAL_H_

class Rational
{
public:
    int GetNumerator();
    int GetDenominator();
    void SetNumerator(const int num);
    void SetDenominator(const int den);

private:
    int mNumerator;
    int mDenominator;
};

#endif
```

```C++
/* Rational.cpp */
#include"Rational.h"

int Rational::GetNumerator()
{
    return mNumerator;
}
    
int Rational::GetDenominator()
{
    return mDenominator;
}

void Rational::SetNumerator(const int num)
{
    mNumerator = num;
}
    
void Rational::SetDenominator(const int den)
{
    mDenominator = den;
}
```

或許到這裡會有點疑惑，為甚麼只是這樣簡單的傳回值跟設定值也要用 member functions 來包裝  
其實這要看我們在設計這個 abstract data type 時的想法是如何，以有理數來說，我們不希望分母是`0`  

```C++
class Rational
{
public:
    int mNumerator;
    int mDenominator;
};

Rational a;
a.mNumerator = 1;
a.mDenominator = 0;
```

上面這種寫法顯然很難阻止分母被設為`0`的狀況  
如果想要禁止分母為`0`，就必須在每次使用到該 data member 的時候都進行檢查  

不過要是透過上面的寫法，我們只需要簡單地作如下的更動  
就可以在外部要改動分母時，先確認是不是`0`以後才設定`mDenominator`  

```C++
void Rational::SetDenominator(const int den)
{
    if(den == 0)
    {
        // TODO
    }
    
    mDenominator = den;
}
```

但這並不代表我們永遠都要把 data members 設為`private`，而是先構思該 abstract data type 是如何被使用  
再進行設計與實作，並盡量把 interface 設計地不容易被錯誤使用  

## Constructor, Default Constructor, Member Initializer List

在`class`中，constructor 是一種 special member functions，他在物件需要被初始化時被呼叫  
而其功用就是初始化該物件的內容  

而 constructor 的宣告如下  

```C++
class_name( parameter_list );
```

於是我們可以進一步擴充`Rational`  

```C++
#include<cassert>

class Rational
{
public:

    Rational(const int n, const int d)
    {
        SetNumerator(n);
        SetDenominator(d);
    }
    
    Rational(const int n)
    {
        SetNumerator(n);
        SetDenominator(1);
    }

    int GetNumerator()
    {
        return mNumerator;
    }
    
    int GetDenominator()
    {
        return mDenominator;
    }
    
    void SetNumerator(const int num)
    {
        mNumerator = num;
    }
    
    void SetDenominator(const int den)
    {
        // 如果 den 為 0，因為條件句為 false，會輸出錯誤訊息以後並結束程式
        assert(den != 0);
        mDenominator = den;
    }

private:
    int mNumerator;
    int mDenominator;
};

Rational a(8, 7);
Rational b(174);
// Rational c;
```

其中首先可以注意到，constructor 與其他 member functions 一樣，都適用於 function overloading  
而在變數被定義的當下，透過後面使用`()`或是`{}`並填入正確的引數，就可以讓編譯器找到符合的 constructor  

其中，以這個 constructor 來說  

```C++
Rational(const int n)
{
    SetNumerator(n);
    SetDenominator(1);
}
```

也可以寫成這樣，這種語法稱作 member initializer list  

```C++
Rational(const int n)
    :mNumerator(n), mDenominator(1)
{
}
```

用 member initializer list 的好處是他相當於 direct initialization  
在此先讓我們先用別的例子，等等再回頭來說為甚麼這個例子我們不使用這個語法  

```C++
#include<string>

class FooA
{
public:
    FooA(const std::string& str)
    {
        mStr = str;
    }
    
private:
    std::string mStr;
};

class FooB
{
public:
    FooB(const std::string& str)
        :mStr(str)
    {
    }
    
private:
    std::string mStr;
};

FooA a(std::string("A"));
FooB b(std::string("B"));
```

以此例來說，`a`裡面的`mStr`會先呼叫 default constructor 初始化自己以後  
再執行`mStr = str;`這行，利用 copy assignment operator 把`str`的內容複製給`mStr`  

但是`b`裡面的`mStr`會直接使用 copy constructor 將`str`的內容複製到`mStr`  
比起`a`來說，少了`mStr`先初始化自己的過程  

另外像是`const`變數或是沒有 default constructor 之類的物件，只能透過 member initializer list 初始化  
如下面的`FooD`就是錯誤的語法  

```C++
class FooC
{
public:
    FooC(const int n)
        :mNum(n)
    {
    }

private:
    const int mNum;
};

/*
class FooD
{
public:
    FooD(const int n)
    {
        mNum = n;
    }

private:
    const int mNum;
};
*/

class FooE
{
public:
    FooE(const int n)
        :mFoo(n)
    {
    }

private:
    FooC mFoo;
};

FooC c(0);
// FooD d(9487);
FooE e(100);
```

還有要注意的是 member initializer list 的填寫順序與初始化順序無關  
初始化的順序是依照 data members 在`class`定義時的順序  
所以通常會建議 member initializer list 的初始化順序要與定義 data members 時的順序一致  

```C++
class FooF
{
public:
    FooF(const int n)
        :mB(n), mA(mB) // mA 的值是 indeterminate value
    {
    }

private:
    int mA;
    int mB;
};

int main()
{
    FooF f(9487);

    return 0;
}
```

看完了 member initializer list，或許讀者會疑問為甚麼`Rational`不選擇用此方式初始化？  
在這裡，筆者認為因為`SetDenominator`裡面已經有不只單純賦值的動作  
為了避免「檢查分母不為`0`」的部分重複在多個地方，於是在這裡選擇呼叫 member functions  

> Note:  
> 因為`mNumerator`與`mDenominator`的型別是`int`，所以在不寫 member initializer list 時  
> 並不會先被清為`0`，所以不用擔心總共被賦值兩次  

再次回到本節一開始的程式碼  
可以看到有行被註解的程式碼，`// Rational c;`，如果不使這句成為註解，那麼將會產生編譯錯誤  
原因出自於`Rational`沒有任何 constructor 是可以不填寫任何引數就可以被呼叫的  
而這種 constructor 又被稱為 default constructor  

到這裡，或許讀者會疑問，在一開始還沒有寫上我們自己的 constructor 以前，為甚麼就沒問題？  
因為一旦我們定義了自己的 constructor，編譯器就不會隱式地為我們產生一個 default constructor  
類似的狀況也發生在有些 special member functions  

如果我們沒有自己定義的 default constructor，又希望保留編譯器替我們產生的 default constructor  
可以先寫出 default constructor 的宣告以後，在後面接上`= default`來達成此目的  
若是想明確地禁止，則是使用`= delete`  

之後若其他的 special member functions 也是有編譯器會隱式地產生的狀況，也可以用同樣方法保留或是禁止  

```C++
struct FooA
{
    FooA(int n)
    {
    }
    
    FooA() = default;
};

struct FooB
{
    FooB(int n)
    {
    }
    
    FooB() = delete;
};

FooA a;
FooA b(0);

// FooB c;
FooB d(0);
```

但是在這裡，如果我們希望允許 default constructor 能初始化成有意義的值的話，不能使用`= default`  
因為若是使用`= default`，承上面的 Note，`Rational`的 data members 都會被賦予 indeterminate value  
與其可能讓使用者做出錯誤的舉動，不如禁止此行為，或是自己明確地定義 default constructor  

```C++
#include<cassert>

class Rational
{
public:

    Rational()
    {
        SetNumerator(0);
        SetDenominator(1);
    }

    Rational(const int n, const int d)
    {
        SetNumerator(n);
        SetDenominator(d);
    }
    
    Rational(const int n)
    {
        SetNumerator(n);
        SetDenominator(1);
    }

    int GetNumerator()
    {
        return mNumerator;
    }
    
    int GetDenominator()
    {
        return mDenominator;
    }
    
    void SetNumerator(const int num)
    {
        mNumerator = num;
    }
    
    void SetDenominator(const int den)
    {
        // 如果 den 為 0，因為條件句為 false，會輸出錯誤訊息以後並結束程式
        assert(den != 0);
        mDenominator = den;
    }

private:
    int mNumerator;
    int mDenominator;
};

Rational a(8, 7);
Rational b(174);
Rational c;
```

## Delegating Constructor, Converting constructor

















