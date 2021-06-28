---
title: Java Nested Classes
published: true
categories: java 
tags: [java, nested class, inner class, static nested class, non-static nested class]
---

원문: [Nested Classes (The Java Tutorials > Learning the Java Language > Classes and Objects)](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)
### 1. 개요  
- 다른 class 안에 정의된 class를 nested class 라고 한다.  

```java
class OuterClass {
    ...
    class NestedClass {
        ...
    }
}
```

- Nested class 는 다음과 같이 두 종류로 나눌 수 있다.  
Non-static nested class (inner class 라고 한다.)  
static nested class   

- Nested class 는 enclosing class의 멤버이다.  
- Non-static nested class (inner class) 는 enclosing class의 다른 멤버들(private 으로 선언되었다 하더라도)에 대한 접근 권한을 가진다.
- OuterClass의 멤버로써 nested class 는 private, public, protected, package private 으로 선언될 수 있다. (OuterClass는 public 이나 package private 으로만 선언가능하다.)  

### 2. Nested Class 사용 이유  
- 한 곳에서만 사용되는 class들을 논리적으로 그룹화하는 방법이다.  
Class 가 단지 하나의 다른 class 에만 사용된다면 그 class 에 포함시켜 관리하는 것이 논리적이다.  
"helper classes"를 중첩(nesting)하면 package를 좀 더 능률적으로 만들어준다.  
- encapsulation 을 증가시킨다.  
Top-level의 A, B class 가 있다고 가정하자.  
B는 A의 private으로 선언된 멤버에 대한 접근 권한이 필요하다.  
Class B를 class A에 감춤으로써 A의 멤버는 private 으로 선언될 수 있고 B는 이들을 접근할 수 있다.  
추가로 B 자체는 외부로부터 숨겨질 수 있다.  
- 좀 더 읽고 유지하기 좋은 코드를 유도할 수 있다.  
Top-level class에 작은 class들을 nesting하는 것은 코드를 사용되는 곳에 더 가깝게 위치하도록 한다.  

### 3. Inner Classes  
- Instance methods와 variables와 같이 inner class는 enclosing class의 instance와 연계되며 이 object의 method와 field에 대한 직접적인 접근권한을 가진다.  
- 또한 inner class는 instance와 연계되므로 자체적으로 static member를 정의할 수 없다.  
- InnerClass의 instance는 OutClass의 instance 안에만 존재할 수 있으며 enclosing instance의 method와 field에 대한 직접 접근 권한을 가진다.  
- Inner class를 instance화 하기 위해서는 우선 outer class를 instance화해야 한다.  

```java
OuterClass outerObject = new OuterClass();
OuterClass.InnerClass innerObject = outerObject.new InnerClass();
```

- Inner class에는 local class와 anonymous class가 있다.  

### 4. Static Nested Classes  
- Class method와 variable와 같이 static nested class는 outer class와 연계되어 있다.  
- Static class method처럼 static nested class는 enclosing class에 정의되어있는 instance variables나 methods를 직접 참조할 수 없다. : 오직 object reference를 통해서만 가능하다. 
- Static nested class는 결과적으로 다른 top-level class에 있는 top-level class처럼 동작한다.  
- Static nested class는 top-level class와 같은 방식으로 초기화된다.  

```java
StaticNestedClass staticNestedObject = new StaticNestedClass();
```

### 5. Inner Class와 Nested Static Class 
- 다음 예제는 OuterClass와 TopLevelClass가 나오며 inner class(InnerClass), nested static class(StaticNestedClass), top-level class(TopLevelClass)가 접근할 수 있는 OutClass의 멤버를 보여준다.  

```java
//OuterClass.java
public class OuterClass {
    String outerField = "Outer field";
    static String staticOuterField = "Static outer field";

    class InnerClass {
        void accessMember() {
            System.out.println(outerField);
            System.out.println(staticOuterField);
        }
    }

    static class StaticNestedClass {
        void accessMember(OuterClass outer) {
            // Compiler error: Cannot make a static reference to the non-static
            //     field outerField
            // System.out.println(outerField);
            System.out.println(outer.outerField);
            System.out.println(staticOuterField);
        }
    }

    public static void main(String[] args) {
        System.out.println("Inner class:");
        System.out.println("------------");
        OuterClass outerObject = new OuterClass();
        OuterClass.InnerClass innerObject = outerObject.new InnerClass();
        innerObject.accessMembers();

        System.out.println("\nStatic nested class:");
        System.out.println("--------------------");
        StaticNestedClass staticNestedObject = new StaticNestedClass();        
        staticNestedObject.accessMembers(outerObject);
        
        System.out.println("\nTop-level class:");
        System.out.println("--------------------");
        TopLevelClass topLevelObject = new TopLevelClass();        
        topLevelObject.accessMembers(outerObject); 
    }

//TopLevelClass.java
public class TopLevelClass {

    void accessMembers(OuterClass outer) {     
        // Compiler error: Cannot make a static reference to the non-static
        //     field OuterClass.outerField
        // System.out.println(OuterClass.outerField);
        System.out.println(outer.outerField);
        System.out.println(OuterClass.staticOuterField);
    }  
}

//Output
Inner class:
------------
Outer field
Static outer field

Static nested class:
--------------------
Outer field
Static outer field

Top-level class:
--------------------
Outer field
Static outer field

```
- Static nested class StaticNestedClass는 outerFiled에 직접 접근할 수 없다.
이유는 enclosing class(OuterClass)의 instance variable이기 때문이다. 
- 비슷하게 top-level class TopLevelClass도 outerField에 직접 접근할 수 없다.  

### 6. Shadowing  
- 특정 영역(inner class나 method definition 같은)에서의 타입 선언(member variable이나 parameter naem)이 enclosing scope내의 다른 선언과 같은 이름을 가지는 경우 그 선언은 enclosing scope의 선언을 shadow하게 된다. shadow된 선언을 이름만으로 참조할 수 없다.  
- 다음 예제(ShadowTest)는 이점을 보여준다.   

```java
public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}

//Result
x = 23
this.x = 1
ShadowTest.this.x = 0
```

- 이 예제는 세 개의 변수 x를 정의한다. 
the member variable of the class ShadowTest,
the member variable of the inner class ShadowTest
the parameter in the method methodInFirstLevel
- method methodInFirstLevel의 parameter로 정의된 x는 FirstLevel의 변수를 shadow한다. 
- 결과적으로 method methodInFirstLevel안에서 변수x를 사용할 때 method parameter를 참조한다. 
inner class FirstLevel의 member 변수를 참조하기 위해서는 enclosing scope를 나타내는 this 키워드를 사용한다. 
- 클래스 명을 사용하여 더 큰 scope에 있는 member variable을 참조한다. 
예를 들어, 다음 statement는 class의 member variable을 접근한다. 
System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);

### 7. Serialization  
- local, anonymous class를 포함한 inner class의 직렬화는 권장되지 않는다.  
- Java compiler가 inner class와 같은 특정 construct(class, method, field, 소스상의 construct와 연관되지 않는 다른 construct)를 컴파일 할 때 synthetic construct를 만든다. Synthetic construct는 java compiler가 JVM의 변환없이 새로운 java language를 실행하도록 한다. 
- 서로 다른 java compiler implementation들 간에 synthetic construct는 매우 다양할 수 있다. ( .class 파일이 implementation간에 다양할 수 있다는 말이다.)
- 결론적으로 서로 다른 JRE implementation에서 inner class를 serialize하고 deserialize한다면 compatibility issue가 발생할 것이다. 

