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

在測試依賴於全域變數時，本質上有三個選擇：
1. 想辦法讓它依賴的全域變數，在測試期間具有另一種行為 （ex: 不同的env檔）
2. 利用連接器，連接到另一個全域變數定義
3. 將其封裝，進而可以進行解耦

```c++
bool AGG230_activeframe[AGG230_SIZE];
bool AGG230_suspendedframe[AGG230_SIZE];

void AGGController::suspend_frame()
{
    frame_copy(AGG230_suspendedframe,
            AGG230_activeframe);
    clear(AGG230_activeframe);
    flush_frame_buffers();
}

void AGGController::flush_frame_buffers()
{
    for (int n = 0; n < AGG230_SIZE; ++n) {
        AGG230_activeframe[n] = false;
        AGG230_suspendedframe[n] = false;
    }
}

```

**suspend_frame**函數需要存取**AGG230_activeframe**跟**AGG230_suspendedframe**這兩個全域陣列。

乍看之下，可以將這兩個陣列做成**AGGController**的成員變數，然而其實行不通。
因為還有其他類別也要用到這兩個陣列。

你可能會想到**Parameterize Method （參數化方法）**，
將它們作為參數傳遞給**suspend_frame**函數。

實際上這樣做的問題是，若**suspend_frame**呼叫了某個函數，
而後者也使用全域變數的話，我們就必須同樣地將該變數傳遞給它。

另一個選擇是將全域陣列傳遞給**AGGController**的建構子。
這麼做是可行的，也可以順便檢查其他用到的地方。

如果發現每次都是兩兩一起被使用，可考慮將它們綁在一起。

> If several globals are always used or are modified near each other, they belong in the same class.

應付這種情況的最佳方法，就是觀察資料。

> When naming a class, think about the methods that will eventually reside on it. The name should be good, but it doesn’t have to be perfect. Remember that you can always rename the class later.

在上例中，我期望**frame_copy**和**clear**能被移至我們將要建立的新類別中。
我們可以命名為**Frame**，每個**Frame**都包含活動緩衝區與懸置緩衝區。

>  The class name that you find might already be in use. If so, consider whether you can rename whatever is using that name.

```c++
class Frame
{
public:
    // declare AGG230_SIZE as a constant
    enum { AGG230_SIZE = 256 };

    bool AGG230_activeframe[AGG230_SIZE];
    bool AGG230_suspendedframe[AGG230_SIZE];
};
```

此處保留兩個陣列的原始名稱，這是為了簡化後面的步驟。

```c++
// 宣吿Frame類別的全域物件
Frame frameForAGG230;

// 註解原本的宣告
// bool AGG230_activeframe[AGG230_SIZE];
// bool AGG230_suspendedframe[AGG230_SIZE];
```

接著透過編輯器，找到每一行出錯的程式碼。
將裡面用到這兩個陣列的宣告，加上**frameForAGG230**的前綴。

```c++
void AGGController::suspend_frame()
{
    frame_copy(frameForAGG230.AGG230_suspendedframe,
                frameForAGG230.AGG230_activeframe);
    clear(frameForAGG20.AGG230_activeframe);
    flush_frame_buffer();
}
```

> Referencing a member of a class rather than a simple global is only the first step. Afterward, consider whether you should use **Introduce Static Setter （靜態設置方法）** , or parameterize the code using **Parameterize Constructor （參數化建構子）** or **Parameterize Method （參數化方法）**.

一旦實作了將**Frame**傳遞進**AGGController**之後，
我們就可以做一點小小的重新命名，讓程式碼變得更清晰。

```c++
class Frame
{
public:
    enum { BUFFER_SIZE = 256 };
    bool activebuffer[BUFFER_SIZE];

    bool suspendedbuffer[BUFFER_SIZE];
};

Frame frameForAGG230;

void AGGController::suspend_frame()
{
    frame_copy(frame.suspendedbuffer, frame.activebuffer);

    clear(frame.activeframe);
    flush_frame_buffer();
}
```

>  When you use Encapsulate Global References, start with data or small methods. More substantial methods can be moved to the new class when more tests are in place.

除了對全域變數可以用此技術外，
面對C++的非成員函數 (global functions)也可以做同樣的事。

```c++
void ColumnModel::update()
{
    alignRows();
    Option resizeWidth = ::GetOption("ResizeWidth");
    if (resizeWidth.isTrue()) {
        resize();
    } else {
        resizeToDefault();
    }
}
```

我們一樣可以**Parameterize Method （參數化方法）**和**Extract and Override Getter （提取並覆寫獲取方法）**。
但如果這些呼叫跨越多個方法多個類別，則使用 **Encapsulate Global References （封裝全域參照）**就更乾淨些。

```c++
class OptionSource
{
public:
    virtual ~OptionSource() = 0;
    virtual Option GetOption(const string& optionName) = 0;
    virtual void SetOption(const string& optionName,
                            const Option& newOption) = 0;
};
```

該類別包含我們所需的**global functions**的抽象版本。
(ʕ •ᴥ•ʔ：其實就是interface啦)

```c++
class ProductionOptionSource : public OptionSource
{
public:
    Option GetOption(const string& optionName);
    void SetOption(const string& optionName,
                const Option& newOption);
};

Option ProductionOptionSource::GetOption(
    const string& optionName)
{
    ::GetOption(optionName);
}

void ProductionOptionSource::SetOption(
    const string& optionName,
    const Option& newOption)
{
    ::SetOption(optionName, newOption);
};
```

> To encapsulate references to free functions, make an interface class with fake and production subclasses. Each of the functions in the production code should do noth- ing more than delegate to a global function.

這個重構引入了**seam （接縫）**。

之後我們便可以對目標類別進行參數化，
使其接受一個**OptionSource**物件。

於測試時傳遞偽**OptionSource**物件，並在產品程式碼傳遞真**OptionSource**物件。

### Steps
To Encapsulate Global References, follow these steps:
1. Identify the globals that you want to encapsulate.
2. Create a class that you want to reference them from.
3. Copy the globals into the class. If some of them are variables, handle their initialization in the class.
4. Comment out the original declarations of the globals.
5. Declare a global instance of the new class.
6. **Lean on the Compiler （依靠編輯器）** to find all the unresolved references to the old globals.
7. Precede each unresolved reference with the name of the global instance of the new class.
8. In places where you want to use fakes, use **Introduce Static Setter （靜態設置方法）**, **Parameterize Constructor （參數化建構子）**, **Parameterize Method （參數化方法）** or **Replace Global Reference with Getter （獲取方法替換全域參照）**.

---

## Expose Static Method （暴露靜態方法）

有時沒辦法在測試控制工具下實例化的類別，可以採用一種技術。
假設有一個方法，它不使用實例變數或其他方法，就可以將它**設成靜態**的。

```java
class RSCWorkflow
{
    ...
    public void validate(Packet packet)
            throws InvalidFlowException {
        if (packet.getOriginator().equals( "MIA")
                || packet.getLength() > MAX_LENGTH
                || !packet.hasValidCheckSum()) {
            throw new InvalidFlowException();
        }
        ...
    }
    ...
}
```

在上述例子中，我們會發現validate的方法，大都來自Packet，
或許將此方法移到Packet會是不錯選擇，但就現階段而言風險還是太大點。

> 在沒有測試的情況下解依賴，盡可能地進行簽章保持。
> 對整個方法進行剪下/貼上可以降低引入錯誤的可能性。

由於validate**沒有依賴任何實例變數或方法**。我們可以將它改為**公有靜態**的。

（註：在某些語言中，類別的靜態部分並不屬於該類別，而是隸屬於另一個類別，有時稱為元類別。）

```java
public class RSCWorkflow {
    public void validate(Packet packet)
            throws InvalidFlowException {
        validatePacket(packet);
    }

    public static void validatePacket(Packet packet)
            throws InvalidFlowException {
        if (packet.getOriginator() == "MIA"
                || packet.getLength() <= MAX_LENGTH
                || packet.hasValidCheckSum()) {
            throw new InvalidFlowException();
        }
        ...
    }
    ...
}
```

> 在某些語言中可以直接將原方法設為靜態即可。但在有些語言中這樣會招來編譯警告。

### Steps
To **Expose Static Method （暴露靜態方法）**, follow these steps:
1. Write a test that accesses the method that you want to expose as a public
static method of the class.
2. Extract the body of the method to a static method. Remember to **Preserve Signatures （簽章保持）**. You’ll have to use a different name for the method. Often you can use the names of parameters to help you come up with a new method name. For example, if a method named validate accepts a Packet, you can extract its body as a static method named validatePacket.
3. Compile.
4. If there are errors related to accessing instance data or methods, take a look at those features and see if they can be made static also. If they can, make them static so that the system will compile.

---

## Extract and Override Call （提取並覆寫呼叫）

當我們遇到想要替換掉的方法呼叫時，若能解開依賴，就能防止測試帶來的副作用。

```c#
public class PageLayout
{
    private int id = 0;
    private List styles;
    private StyleTemplate template;
    ...
    protected void rebindStyles() {
        styles = StyleMaster.formStyles(template, id);
       ...
    }
    ...
}
```

上述例子中，rebindStyles依賴了SysleMaster的formStyles，讓我們試著解開它。

```c#
public class PageLayout {
    private int id = 0;
    private List styles;
    private StyleTemplate template;
    ...
    protected void rebindStyles() {
        styles = formStyles(template, id);
    ...
    }

    protected List formStyles(StyleTemplate template, int id) {
        return StyleMaster.formStyles(template, id);
    }
    ...
}
```

一旦有了formStyles方法，我們便可以覆寫它來解開依賴了。

```c#
public class TestingPageLayout extends PageLayout {
    protected List formStyles(StyleTemplate template,
                            int id) {
        return new ArrayList();
    }
    ...
}
```

### Steps
To **Extract and Override Call （提取並覆寫呼叫）**, follow these steps:
1. Identify the call that you want to extract. Find the declaration of its method.
Copy its method signature so that you can **Preserve Signatures （簽章保持）** .
2. Create a new method on the current class. Give it the signature you’ve
copied.
3. Copy the call to the new method and replace the call with a call to the new method.

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

