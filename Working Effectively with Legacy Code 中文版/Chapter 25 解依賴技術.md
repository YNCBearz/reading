# Chapter 25 - Dependency-Breaking Techniques（解依賴技術）

## Adapt Parameter （參數適配）

>  Use *Adapt Parameter* when you can’t use *Extract Interface (362)* on a parameter’s class or when a parameter is difficult to fake.

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

如果我們改用**ParameterSource**，則**populate**方法的邏輯就跟特定的參數源分離開來。程式碼不再依賴特定的**J2EE**介面。

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

