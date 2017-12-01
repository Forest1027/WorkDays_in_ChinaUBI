# 2017/12/1
## trasient关键字

[source](https://www.cnblogs.com/lanxuezaipiao/p/3369962.html)

**1.transient的概况**

我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

**总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。**

*通过代码中的方法测试序列化前和序列化之后属性值的变化，运行结果发现被transient修饰后的数据不会序列化到文件中*

*代码如下：*

    public class Demo1 {
        public static void main(String[] args) {
            User user = new User(1234L, "name", 18);
            System.out.println("read before Serializable: ");
            System.out.println("username: " + user.getName());
            System.err.println("age: " + user.getAge());
            try {
                ObjectOutputStream os = new ObjectOutputStream(
                        new FileOutputStream("E:\\forest's demo\\demo1.txt"));
                os.writeObject(user); // 将User对象写进文件
                os.flush();
                os.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
            try {
                ObjectInputStream is = new ObjectInputStream(new FileInputStream(
                        "E:\\forest's demo\\demo1.txt"));
                user = (User) is.readObject(); // 从流中读取User的数据
                is.close();
                System.out.println("\nread after Serializable: ");
                System.out.println("username: " + user.getName());
                System.err.println("age: " + user.getAge());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }


    @ToString
    public class User implements Serializable{
        private Long id;
        public String name;
        private transient Integer age;
        public User() {
        }
        public Long getId() {
            return id;
        }
        public void setId(Long id) {
            this.id = id;
        }
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public Integer getAge() {
            return age;
        }
        public void setAge(Integer age) {
            this.age = age;
        }
    }


> **小结**
> 1. 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
> 2. transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口
> 3. 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。
> **特别注意**：
> 第三点可能有些人很迷惑，因为发现在User类中的username字段前加上static关键字后，程序运行结果依然不变，即static类型的username也读出来为“Alexia”了，这不与第三点说的矛盾吗？实际上是这样的：第三点确实没错（一个静态变量不管是否被transient修饰，均不能被序列化），反序列化后类中static型变量username的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的

*“特别注意”的代码验证如下：*

*运行结果发现，序列化前的静态变量不会序列化到文件中，因为最后打印出来的静态变量是反序列化前更新了的值。也就是说打印出来的静态变量不是从文件中读取的值*

    public class Demo1 {
        public static void main(String[] args) {
            User user = new User();
            user.setAge(18);
            user.setName("name");
            System.out.println("read before Serializable: ");
            System.out.println("username: " + user.getName());
            System.err.println("age: " + user.getAge());
            try {
                ObjectOutputStream os = new ObjectOutputStream(
                        new FileOutputStream("E:\\forest's demo\\demo1.txt"));
                os.writeObject(user); // 将User对象写进文件
                os.flush();
                os.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
            try {
                //在反序列前改变静态变量的值
                User.name="eman";
                ObjectInputStream is = new ObjectInputStream(new FileInputStream(
                        "E:\\forest's demo\\demo1.txt"));
                user = (User) is.readObject(); // 从流中读取User的数据
                is.close();
                System.out.println("\nread after Serializable: ");
                System.out.println("username: " + user.getName());
                System.err.println("age: " + user.getAge());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

     @ToString
    public class User implements Serializable{
        private Long id;
        public static String name; //用static修饰变量
        private transient Integer age;
        public User() {
        }
        public Long getId() {
            return id;
        }
        public void setId(Long id) {
            this.id = id;
        }
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public Integer getAge() {
            return age;
        }
        public void setAge(Integer age) {
            this.age = age;
        }
    }

**2.transient的特别情况**

不是加了transient关键字就一定不会被序列化了。

我们知道在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关（也就是说也会被序列化）。

## how to get started with open source projects in github

**comprehensive introduction**
[source](https://github.com/collections/choosing-projects)

**9 streps** 
[source](https://maurobringolf.ch/2017/07/open-source-9-steps-to-my-first-feature-contribution-in-babel/)

* Understand what the project does and explain how it works in a few sentences.
* Learn the APIs of the project and preferably use them in your own work.
* Read all documentation available.
* Look at the different modules and get a sense of how they interact.
* Follow current issues and pull requests on GitHub.
* Decide on one area to go deeper and start looking at test cases of these modules.
* Contribute documentation and tests. Fix typos and increase test coverage by 0.0x%.
* Focus on beginner issues and see how they are solved.
* Fix issues yourself.

**good chicken soup**
[source](https://medium.freecodecamp.org/a-beginners-very-bumpy-journey-through-the-world-of-open-source-4d108d540b39)

It’s okay to not know everything, and take one step at a time to learn something new.

