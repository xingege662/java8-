[TOC] 
## 行为参数化的3中方式(策略模式)
1.  类
2.  匿名类
3.  lamada
## 函数式接口
函数式接口就是只定义一个抽象方法的接口。

Lambda表达式允许你直接内联，为函数式接口的抽象方法提供实现，并且将整个表达式座位函数式接口的一个实例。

## java.util.function包中引入了的函数式接口
1. Predicate

```
Interface Predicate<T>

Type Parameters:
T - the type of the input to the predicate

Functional Interface:
This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.
```
抽象方法：
```
boolean	test(T t)
Evaluates this predicate on the given argument.
```
Predicate<T>接口定义了一个名叫test的抽象方法,接收泛型T对象，返回一个boolean。

```
public class Main {

    public static void main(String[] args) {
        Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
        filter(listOfString, nonEmptyStringPredicate);
    }

    public static <T> List<T> filter(List<T> list, Predicate<T> p){
        List<T> results = new ArrayList<T>();
        for(T s : list){
            if(p.test(s)){
                results.add(s);
            }
        }
        return results;
    }
}
```
2. Consumer

```
Interface Consumer<T>

Type Parameters:
T - the type of the input to the operation

All Known Subinterfaces:
Stream.Builder<T>

Functional Interface:
This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.

```
抽象方法

```
void	accept(T t)
Performs this operation on the given argument.
```
Consumer<T>定义了一个名叫accept的抽象方法，它接受泛型T对象，没有返回(void)。如果需要访问类型T的对象，并对其执行某些操作，就可以使用这个接口。


```
  public static <T> void forEach(List<T> list,Consumer<T> consumer){
        for(T i:list){
            consumer.accept(i);
        }
    }
    
   forEach(Arrays.asList(1,2,3,4,5),(Integer i) -> System.out.println(i));
```
3. Interface Function<T,R>


```
Type Parameters:
T - the type of the input to the function
R - the type of the result of the function

All Known Subinterfaces:
UnaryOperator<T>

Functional Interface:
This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.
```
抽象方法:

```
R	apply(T t)
Applies this function to the given argument.
```
Function<T,R>接口定义了一个叫做apply的方法，它接受一个泛型T的对象，并返回一个泛型R的对象。如果要定义一个lambda，将输入对象的信息映射到输出，就可以使用这个接口。


```
  public static <T,R> List<R> map(List<T> list, Function<T,R> f){
            List<R> result = new ArrayList<>();
            for(T s:list){
                result.add(f.apply(s));
            }
            return  result;
    }
    
 List<Integer> l = map(Arrays.asList("lambdas","in","action"),(String s)->s.length());
    }
```
## 异常，lambda，还有函数式接口

任何函数式接口都不允许抛出受检异常(checked  exception).如果需要Lambda表达式来抛出异常，有两种办法：
1. 定义字一个自己的函数式接口，并声明受检异常。
2. 把lambda包在一个try/catch中。

```
@FunctionalInterface
public interface BufferedReaderProcessor{
    String process(BufferdReader b) throws IOException
}
```

```
Function<BufferedReader,String> f = (BufferedReader b) ->{
    try{
        return b.readLine();
    }catch(IOException e){
        throw new RuntimeException(e);
    }
}
```
## Lambda表达式使用局部变量

lambda表达式不仅能用到主体里的函数。还允许使用自有变量(不是参数，而是在外层作用域中定义的变量)，就像匿名类一样。他们被称作捕获Lambda
    
```
int portNumber = 1337;
Runable r = () -> System.out.println(portNumber);
```
但是,Lambda可以没有限制的捕获(也就是在其主题中引用)实例变量和静态变量。但是局部变量必须显示的声明为final。

## 闭包
    
Lambda和匿名类可以做类似于闭包的事情：他们可以作为参数传递给方法，并且可以访问其作用域之外的变量。但是有一个显示：他们不能修改定义Lambda的方法的局部变量的内容。
## 方法引用
先前

```
inventory.sort((Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```
之后(使用方法引用后java.util.Comparator.compareing):

```
inventory.sort(comparing(Apple::getWeight))
```
如何构建方法引用：方法引用主要有3类
1. 指向静态方法的方法引用(例如Integer的parseInt方法)写作Integer::parseInt
2. 指向任意类型实例方法的方法引用(例如String的length方法,写作String::length)
3. 指向现有对象的实例方法的方法引用(假设你有一个局部变量expensiveTransaction用于存放Transaaction类型的对象，他支持实例方法getValue，那么你就可以写expensiveTransaction::getValue)。
![image](http://opy4iwqsf.bkt.clouddn.com/WX20170620-142101@2x.png)

## 构造函数引用
对于一个现有的构造函数，可以利用它的名称和关键字new来创建它的一个引用:ClassName::new。它的功能与指向静态方法的引用类似。列入，假设有一个构造函数没有参数。它适合Supplier的签名()->Apple。
    
```
Suppier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
```
这就等价于：

```
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();

```
如果构造函数的签名是Apple(Integer weight),那么它就适合Function接口的签名，于是你可以这样写：

```
Function<Integer,Apple> c2 = Apple::new;
Apple a2 = c2.apply(110);
```
等价于：

```
Function<Integer,Apple> c2 = (weight) ->new Apple(weight);
Apple a2 = c2.apply(110);
```
## 谓词复合
谓词接口包括三个方法：negate，and和or,让你可以重用已有的Predicatre来创建更复杂的谓词。
![image](http://opy4iwqsf.bkt.clouddn.com/WX20170620-162841@2x.png)

## 函数复合

![image](http://opy4iwqsf.bkt.clouddn.com/WX20170620-163244@2x.png)
