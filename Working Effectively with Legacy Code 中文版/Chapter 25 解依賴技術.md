# Chapter 25 - Dependency-Breaking Techniques（解依賴技術）

## Adapt Parameter （參數適配）

>  Use *Adapt Parameter* when you can’t use *Extract Interface* on a parameter’s class or when a parameter is difficult to fake.

ʕ •ᴥ•ʔ：利用**Wrapper**的概念，將參數封裝。

```java
public class ARMDispatcher
{
    public void populate(HttpServletRequest request) {
        String [] values
            = request.getParameterValues(pageStateName);
        if (values != null && values.length > 0)
        {
            marketBindings.put(pageStateName + getDateStamp(), values[0]);
        }
        ...
    }
    ...
}
```

此處的**HttpServletRequest**為**J2EE**中的介面。
其中宣告了至少23個方法。

這邊的解法一，是去下載一個**J2EE**的**仿物件(mock object)函式庫**。

~~但作者想秀一下自己很厲害。~~

解法二是本節的手法，將**參數類型外覆**起來。

```java
public interface ParameterSource {
    public void getParameterForName(String name);
}

class FakeParameterSource implements ParameterSource {
    public String value;

    public String getParameterForName(String name) {
        return value;
    }
}

class ServletParameterSource implements ParameterSource {
    private HttpServletRequest request;
    public ServletParameterSource(HttpServletRequest request) {
        this.request = request;
    }

    String getParameterForName(String name) {
        String [] values = request.getParameterValues(name);
        if (values == null || values.length < 1)
            return null;
        return values[0];
    }
}
```

從表面上來看，僅僅是為了美觀而美觀。
但若你能夠建立較窄且針對特定需求的介面，程式碼便能充分傳達語義，
而其中也有更佳的接縫。

如果我們改用**ParameterSource**，則**populate**方法的邏輯就跟特定的參數源分離開來。
程式碼不再依賴特定的**J2EE**介面。

> Adapt Parameter is one case in which we don’t **Preserve Signatures**. Use extra care.

此方法有時會引入微小bug，記住我們的目的是使測試到位。
一旦測試到位，我們就更有信心進行侵入式變動。

### Steps
To use **Adapt Parameter**, perform the following steps:
1. Create the new interface that you will use in the method. Make it as sim- ple and communicative as possible, but try not to create an interface that will require more than trivial changes in the method.
2. Create a production implementer for the new interface.
3. Create a fake implementer for the interface.
4. Write a simple test case, passing the fake to the method.
5. Make the changes you need to in the method to use the new parameter.
6. Run your test to verify that you are able to test the method using the fake.

---

## Break Out Method Object （分解出方法物件）

> In a nutshell, the idea behind this refactoring is to move a long method to a new class.

```c++
class GDIBrush
{
public:
    void draw(vector<point>& renderingRoots,
                ColorMatrix& colors,
                vector<point>& selection);
    ...

private:
    void drawPoint(int x, int y, COLOR color);
    ...
};


void GDIBrush::draw(vector<point>& renderingRoots,
                    ColorMatrix& colors,
                    vector<point>& selection)
{
    for(vector<points>::iterator it = renderingRoots.begin();
            it != renderingRoots.end();
            ++it) {
        point p = *it;
        ...
        drawPoint(p.x, p.y, colors[n]);
    }
    ...
}
```

**GDIBrush**的**draw**是個長方法，我們無法輕易地為其編寫測試。
且在測試工具中，也不好建立**GDIBrush**的實例。

第一步就是建立負責畫圖的新類別，我們稱作**Renderer**。
這邊會運用到**Preserve Signatures （保持簽章）**的手法。

```c++
class Renderer
{
    public:
        Renderer(GBIBrush *brush,
            vector<point>& renderingRoots,
            ColorMatrix &colors,
            vector<point>& selection);
        ...
};
```

建立了建構子之後，我們便可以為它的每個參數建立成員變數，並初始化。
為了保持簽章，此處也是透過copy/paste來完成的。

```c++
class Renderer {
    private:
        GDIBrush *brush;
        vector<point>& renderingRoots;
        ColorMatrix& colors;
        vector<point>& selection;

    public:
        Renderer(GDIBrush *brush,
                vector<point>& renderingRoots,
                ColorMatrix& colors,
                vector<point>& selection)
        : brush(brush), renderingRoots(renderingRoots),
        colors(colors), selection(selection)
        {}
};
```

看到這你可能會說，這到底有什麼好處？
我們還是有GDIBrush的參照！

稍安毋躁，馬上你就會發現情況不同。

1、2、3...發現了嗎？

我們接著加入**draw**方法。

```c++
class Renderer {
    private:
        GDIBrush *brush;
        vector<point>& renderingRoots;
        ColorMatrix& colors;
        vector<point>& selection;

    public:
        Renderer(GDIBrush *brush,
                vector<point>& renderingRoots,
                ColorMatrix& colors,
                vector<point>& selection)
        : brush(brush), renderingRoots(renderingRoots),
        colors(colors), selection(selection)
        {}

        void draw();
};
```

我們將原來的**draw**方法，複製進來。
透過 **Lean on the Compiler （編譯器技術）**告訴我們何處需要變動。

```c++
void Renderer::draw() {
    for(vector<points>::iterator it = renderingRoots.begin();
        it != renderingRoots.end();
        ++it) {
        point p = *it;
        ...
        drawPoint(p.x, p.y, colors[n]);
    }
    ...
}
```

要讓編譯通過，我們可以簡單地為**draw**方法所依賴的成員變數，
分別引入一個獲取方法。

實際上**draw**方法只依賴**drawPoint**的私有方法，
因此我們只需要將其改為公有即可。

於是現在，我們可以將**GDIBrush::draw()**委託給**Renderer**的**draw()**了：

```c++
void GDIBrush::draw(vector<point>& renderingRoots,
                    ColorMatrix &colors,
                    vector<point>& selection)
{
    Renderer renderer(this, renderingRoots,
                        colors, selection);
    renderer.draw();
}
```

接著透過**Extract Interface （介面提取）**的手法，我們提出**PointRenderer**介面。

（下略）

隨著時間推移，你可能會對GDIBrush放入測試工具越來越不滿。
一旦GDIBrush被測試覆蓋，你就可以做許許多多的調整。

> 此方法有幾個變形。
> 最簡單的情況是原方法不使用原類別的任何實例成員。
> 再來是只使用了原類別的資料成員，而不使用方法。
> 這時我們可以建立新類別，將被用到的資料成員放入該類別中。
>
> 本節範例是最複雜的情況：被分解出來的方法用到原類別的方法。
> 因此我們最後使用*Extract Interface （介面提取）*，
> 在提出的方法物件與原物件間，建立一定程度的抽象。

### Steps
You can use these steps to do Break out Method Object safely without tests:
1. Create a class that will house the method code.
2. Create a constructor for the class and **Preserve Signatures** to give it an exact copy of the arguments used by the method. If the method uses an instance data or methods from the original class, add a reference to the original class as the first argument to the constructor.
3. For each argument in the constructor, declare an instance variable and give it exactly the same type as the variable. **Preserve Signatures** by copying all the arguments directly into the class and formatting them as instance variable declarations. Assign all of the arguments to the instance variables in the constructor.
4. Create an empty execution method on the new class. Often this method is called run(). We used the name draw in the example.
5. Copy the body of the old method into the execution method and compile to **Lean on the Compiler**.
6. The error messages from the compiler should indicate where the method is still using methods or variables from the old class. In each of these cases, do what it takes to get the method to compile. In some cases, this is as simple as changing a call to use the reference to the original class. In other cases, you might have to make methods public on the original class or introduce getters so that you don’t have to make instance variables public.
7. After the new class compiles, go back to the original method and change it so that it creates an instance of the new class and delegates its work to it.
8. If needed, use **Extract Interface** to break the dependency on the original class.

---

## Definition Completion （定義補全）

有些語言允許你在一個地方宣告類型，然後在另一個地方定義它。

ʕ •ᴥ•ʔ：聽起來像是Interface。

```c++
class CLateBindingDispatchDriver : public CDispatchDriver {
public:
            CLateBindingDispatchDriver ();
    virtual ~CLateBindingDispatchDriver ();

    ROOTID  GetROOTID (int id) const;

    void    BindName (int id,
                    OLECHAR FAR *name);
private:
    CArray<ROOTID, ROOTID& > rootids;
};
```

用戶建立**CLateBindingDispatchDrivers**的物件，
然後呼叫其**BindName**方法來將名字綁定到ID。

我們希望在測試該類別時，以其他種方式綁定，而不是透過原來的**BindName**方法。

可以透過**Definition Completion （定義補全）**的方式，
於測試檔案中包含其標頭檔，然後對目標方法提供另一個定義：

```c++
#include "LateBindingDispatchDriver.h"

CLateBindingDispatchDriver::CLateBindingDispatchDriver() {}

CLateBindingDispatchDriver::~CLateBindingDispatchDriver() {}

ROOTID GetROOTID (int id) const { return ROOTID(-1); }

void BindName(int id, OLECHAR FAR *name) {}

TEST(AddOrder,BOMTreeCtrl)
{
    CLateBindingDispatchDriver driver;
    CBOMTreeCtrl ctrl(&driver);
    ctrl.AddOrder(COrderFactory::makeDefault());
    LONGS_EQUAL(1, ctrl.OrderCount());
}
```

只需在測試中直接定義有關方法，
我們便可以提供只用於測試的方法定義。

可以給那些我們在測試時並不關心的方法，定義一個空的方法體，
也可以定義可用於所有測試的感測方法。

在C/C++中使用**Definition Completion （定義補全）**，
意謂著我們得**為了測試建立單獨的可執行檔案**。
否則替換的定義會跟原有的定義，在連接期產生衝突。

另一個缺點是，目標類別現在有了兩組定義，
可能會使程式碼的維護帶來很大的負擔。

因此，只建議在解開初始依賴時，使用該技術。

### Steps
To use Definition Completion in C++, follow these steps:
1. Identify a class with definitions you’d like to replace.
2. Verify that the method definitions are in a source file, not a header.
3. Include the header in the test source file of the class you are testing.
4. Verify that the source files for the class are not part of the build.
5. Build to find missing methods.
6. Add method definitions to the test source file until you have a complete build.

---

## Encapsulate Global References （封裝全域參照）



---

## Expose Static Method （暴露靜態方法）

---

## Extract and Override Call （提取並覆寫呼叫）

---

## Extract and Override Factory Method （提取並覆寫工廠方法）

---

## Extract and Override Getter （提取並覆寫獲取方法）

---

## Extract Implementer （實作提取）

---

## Extract Interface （介面提取）

---

## Introduce Instance Delegator （引入實例委託）

---

## Introduce Static Setter （引入靜態設置方法）

