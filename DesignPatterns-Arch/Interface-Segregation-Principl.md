# 接口隔离原则

接口隔离原则(Interface Segregation Principle)：**接口分为两种，实例接口（Object Interface）和类接口（Class Interface，用interface定义的接口），这里讲的就是类接口，接口隔离原则定义为：1，客户端不应该依赖它不需要的接口；2，类间的依赖关系应尽量建立在最小接口上**。

接口要尽量细化，接口中的发尽量要少，与单一职责不同的是，单一职责降调的是职责，这是业务逻辑上的划分，比如按照职责在接口中定义10个方法，提供给多个模块使用，各个模块按照权限来访问，单一职责原则来讲是允许的，而接口隔离原则却不忍许这样做，接口隔离原则要求**提供给每个模块的接口都要单一，提供给几个模块就要有几个接口，而不是建立一个庞大的臃肿的接口，容纳所有客户端**。这样客户端就只能感知到与他们业务相关的方法了。

*   接口尽量要小，这是接口隔离的核心，但是小有限度，所以首先满足单一职责的前提下进行接口细化查分
*   接口要高内聚，尽量少的保留public方法，接口是对外承诺，承诺越少，分享越小
*   接口要符合定制服务，比如图书管理系统，有分类查询与综合公查询，用户只能使用分类查询，而管理员都可以，所以用接口隔离，实现定制服务
*   接口的设计是有限度的，接口多，灵活性高，但是复杂性提高，所以要掌握一个度

## 最佳实践

*   一个接口只服务与一个业务逻辑或一个子模块
*   已经被污染的接口尽量去修改，若变更风险大，可使用适配器模式进行转化处理
*   了解环境需求，根据业务进行接口隔离


## 实例

一般我们在使用IO流的时候，在使用完毕之后都需要关闭IO流，而IO的操作都伴随着IOException，每次书写都要使用try-catch:

```
    File file123 = new File("123.txt");

    BufferedWriter bufferedWriter = null;
    try {
        bufferedWriter = new BufferedWriter(new FileWriter(file123, true));
        bufferedWriter.newLine();
        Properties properties = System.getProperties();
        Enumeration<?> enumeration = properties.propertyNames();

        while (enumeration.hasMoreElements()) {
            Object name = enumeration.nextElement();
            bufferedWriter.write(name.toString() +"-----"+ properties.getProperty(name.toString()));
            bufferedWriter.newLine();
        }
        bufferedWriter.flush();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (bufferedWriter != null) {
            try {
                bufferedWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

每次都要书写try-catch是非常繁琐的，幸好Java中有一个Closeable接口，而Java中的IO流都实现了这个接口，所以我们可以书写一个针对Closeable工具类，以避免每次都要写这个烦人的try-catch代码：

```
    public class IOUtils {
        private IOUtils() {
    
    
        }
    
        public static void close(Closeable closeable) {
            if (closeable != null) {
                try {
                    closeable.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                
            }
        }
    }
```

而这里的IOUtils就是运用的接口隔离原则，Closeable只有一个close方法，这就简化了接口的复杂性。IOUtils仅仅知道Closeable的cloase方法，不关心具体的流的类型。


## 实例 2

图书管理系统多个查询方法：但是complexSearch(Map map)值提供给管理员使用，而用户是不能使用的

```
    public interface IBookSearcher {
    
        void searchByAuthor(String author) ;
        void searchByTitle(String title) ;
        void searchByCategory(String category) ;
    
        /**
         * 综合查询
         * @param map
         */
        void complexSearch(Map map);
    
    
    }
```

按照上面的接口设计，就不符合接口隔离原则，一个接口应该只服务于一个子类型或一个模块，所以可以拆分成一下接口：

```
    public interface IcomplexSearcher {
    
        /**
         * 综合查询
         * @param map
         */
        void complexSearch(Map map);
    }
    
    public interface IBookSearcher {
    
        void searchByAuthor(String author) ;
        void searchByTitle(String title) ;
        void searchByCategory(String category) ;
    
    }
```

管理员同时实现两个接口，二用户只实现一个接口。这样就运用了接口管理原则隔离了用户端对综合查询的感知，增强了系统的灵活性，降低了模块之间的耦合性。





