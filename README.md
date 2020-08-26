![](./img/spring-boot.png)

[Java cơ bản](./DOC/java-basic.md)

[Java OOP](./DOC/java-oop.md)

[Code Java nhanh sử dụng Lombok](./DOC/Lombok.md)

------------------
table of contents

Spring boot

1. [Các khái niệm cần nắm](#spring_other)

2. [Cách cài đặt](#spring_install)

3. [Spring boot @Component và @Autowired](#springboot_1)

   3.1. [Annotation @Component](#spring_component)
   
   3.3.2. [Annotation @Autowired](#spring_autowired)

   3.3.3. [Annotation @Primary](#spring_primary)

4. [Spring Bean Life Cycle, @PostConstruct và @PreDestroy](#springboot_2)

   4.1. [@PostConstruct](#spring_postconstruct)

   4.2. [@PreDestroy](#spring_predestroy)

   4.3. [Bean Life Cycle](#spring_beanlifecycle)

5. [Spring boot @Component, @Service và @Repository](#springboot_3)

   5.1. [Kiến trúc trong spring boot](#springboot_architecture)

   5.2. [@Controller vs @Service vs @Repository](#springboot_csr)

6. [Component scan](#springboot_componentscan)

7. [Spring Boot @Configuration và @Bean](#springboot_conf_bean)

8. [Spring Boot Application Config và annotation @Value](#springboot_8)

9. [Spring Boot @Controller, và ví dụ tạo trang web Hello World](#springboot_9)

10. [Giải thích cách Thymeleaf vận hành + Expression + Demo Full](#springboot_10)



------------------


# Learn Spring Boot Framwork

## Các khái niệm cần nắm <a name="java_other"></a>

### Khái niệm tight-coupling (liên kết ràng buộc) và cách loosely coupled (liên kết lỏng lẻo)

`tight-coupling` hay "liên kết ràng buộc" là một khái niệm trong Java ám chỉ việc mối quan hệ giữa các Class quá chặt chẽ. Khi yêu cầu thay đổi logic hay một class bị lỗi sẽ dẫn tới ảnh hưởng tới toàn bộ các Class khác.

`loosely-coupled` là cách ám chỉ việc làm giảm bớt sự phụ thuộc giữa các Class với nhau.

#### Ví dụ

1. Các code level 1

```java
public class BubbleSortAlgorithm{
   public void sort(int[] array) {
      // TODO: Add your logic here
      System.out.println("Đã sắp xếp bằng thuật toán sx nổi bọt");
   }
}

public class VeryComplexService {
   private BubbleSortAlgorithm bubbleSortAlgorithm = new BubbleSortAlgorithm();

   public VeryComplexService(){
   }

   public void complexBusiness(int array[]){
      bubbleSortAlgorithm.sort(array);
      // TODO: more logic here
   }
}
```

Với cách làm ở trên, `VeryComplexService` đã hoàn thiện được nhiệm vụ, tuy nhiên, khi có yêu cầu thay đổi thuật toán sắp xếp sang QuickSort thì nghe vẻ chúng ta sẽ phải sửa lại hoàn toàn cả 2 Class trên.

Ngoài ra `BubbleSortAlgorithm` sẽ chỉ tồn tại nếu `VeryComplexService` tồn tại, vì `VeryComplexService` tạo đối tượng `BubbleSortAlgorithm` bên trong nó (hay nói cách khác là sự sống chết của `BubbleSortAlgorithm` sẽ do `VeryComplexService` quyết định), theo như cách implement này, nó là liên kết rất chặt với nhau.

2. Cách code level 2

```java
public interface SortAlgorithm {
    /**
     * Sắp xếp mảng đầu vào
     * @param array
     */
    public void sort(int array[]);
}

public class BubbleSortAlgorithm implements SortAlgorithm{

    @Override
    public void sort(int[] array) {
        // TODO: Add your logic here
        System.out.println("Đã sắp xếp bằng thuật toán sx nổi bọt");
    }
}


public class VeryComplexService {
    private SortAlgorithm sortAlgorithm;
    public VeryComplexService(){
        sortAlgorithm = new BubbleSortAlgorithm();
    }

    public void complexBusiness(int array[]){
        sortAlgorithm.sort(array);
        // TODO: more logic here
    }
}
```

Với cách làm này, `VeryComplexService` sẽ chỉ quan hệ với một interface `SortAlgorithm`. Với cách này thì mỗi quan hệ giảm bớt sự liên kết, nhưng nó không thay đổi được việc thuật toán vẫn đang là `BubbleSortAlgorithm`.

3. Cách code level 3

```java
public interface SortAlgorithm {
   /**
   * Sắp xếp mảng đầu vào
   * @param array
   */
   public void sort(int array[]);
}

public class BubbleSortAlgorithm implements SortAlgorithm{

   @Override
   public void sort(int[] array) {
      // TODO: Add your logic here
      System.out.println("Đã sắp xếp bằng thuật toán sx nổi bọt");
   }
}

public class QuicksortAlgorithm implements SortAlgorithm {
   @Override
   public void sort(int[] array) {
      // TODO: Add your logic here
      System.out.println("Đã sắp xếp bằng thuật sx nhanh");
   }
}

public class VeryComplexService {
   private SortAlgorithm sortAlgorithm;
   public VeryComplexService(SortAlgorithm sortAlgorithm){
      this.sortAlgorithm = sortAlgorithm;
   }

   public void complexBusiness(int array[]){
      sortAlgorithm.sort(array);
      // TODO: more logic here
   }
}

public static void main(String[] args) {
   SortAlgorithm bubbleSortAlgorithm = new BubbleSortAlgorithm();
   SortAlgorithm quickSortAlgorithm = new QuicksortAlgorithm();
   VeryComplexService business1 = new VeryComplexService(bubbleSortAlgorithm);
   VeryComplexService business2 = new VeryComplexService(quickSortAlgorithm);
}
```

Cách thứ ba này cũng là cách làm phổ biển nhất. Mối liên hệ giữa 2 Class đã "lỏng lẻo" hơn trước rất nhiều. `VeryComplexService` sẽ không quan tâm tới việc thuật toán sắp xép là gì nữa, mà chỉ cần tập trung vào nghiệp vụ. Còn `SortAlgorithm` sẽ được đưa vào từ bên ngoài tùy theo nhu cầu sử dụng.

### Tiêm phụ thuộc (dependency injection) và IoC (Inversion of Control) 

1. DI

> Các class không nên phụ thuộc vào các kế thừa cấp thấp mà nên phụ thuộc vào class abstraction.

`Dependency Injection` là việc các Object nên phụ thuộc vào các Abstract Class và thể hiện chi tiết của nó sẽ được Inject vào đối tượng lúc runtime.

Các cách để Inject dependency vào một đối tượng có thể kể đến như sau:

- Constructor Injection: Cái này chính là ví dụ của mình, tiêm dependency ngay vào Contructor cho tiện.

- Setter Injection: Ồ, sao không chứ 😗 chúng ta học về Setter từ những bài học vỡ lòng rồi, quá hợp lý. Xài girl.setOutfit(new Naked()) 😈

- Interface Injection: Mỗi Class muốn inject cái gì, thì phải implement một Interface có chứa một hàm inject(xx) (Gần như thay thế cho setter ý bạn). Rồi bạn muốn inject gì đó thì gọi cái hàm inject(xx) ra. Cách này hơi dài và khó cho người mới.

2. IoC

> Inversion of Control is a programming principle. flow of control within the application is not controlled by the application itself, but rather by the underlying framework.

Ta định nghĩa trước toàn bộ các `dependency` có trong Project, mô tả nó và tống nó vào 1 cái kho và giao cho một thằng tên là `framework` quản lý. Bất kỳ các `Class` nào khi khởi tạo, nó cần `dependency` gì, thì cái `framework` này sẽ tự tìm trong kho rồi `inject` vào đối tượng thay chúng ta.

## Cách cài đặt <a name="spring_install"></a>

1. Tạo một Spring boot project 

Vào https://start.spring.io/ tạo 1 project . Ở đây sử dụng `maven` và package là `com.example.blog`

Nếu trong `Java` truyền thống, khi chạy cả một project, chúng ta sẽ phải định nghĩa một hàm `main()` và để nó khởi chạy đầu tiên.

Thì **Spring Boot** cũng vậy, chúng ta sẽ phải chỉ cho **Spring Boot** biết nơi nó khởi chạy lần đầu, để nó cài đặt mọi thứ.

Cách thực hiện là thêm annotation `@SpringBootApplication` trên class chính và gọi `SpringApplication.run(BlogApplication.class, args);` để chạy project.

```java
package com.example.blog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BlogApplication {

	public static void main(String[] args) {
		SpringApplication.run(BlogApplication.class, args);
	}
}
```

Một trong những nhiệm vụ chính của Spring là tạo ra một cái Container chứa các Dependency cho chúng ta.

`SpringApplication.run(BlogApplication.class, args);` chính là câu lệnh *để tạo ra* **container**. Sau đó nó *tìm toàn bộ* các **dependency** trong project của bạn và đưa vào đó.

Spring đặt tên cho **container** là ***ApplicationContext***

và đặt tên cho các **dependency** là ***Bean***


## Hướng dẫn @Component và @Autowired <a name="springboot_1"></a>

`@Component` và `@Autowire` là 2 annotation (chú thích) cơ bản trong Spring boot.

### @Component là một Annotation đánh dấu trên các `class` để giúp **Spring** biết đó là một `bean`. <a name="spring_component"></a>

Ví dụ:

Ta có một interface `Keycap`

```java
public interface Keycap {
    public void insert();
}
```

implement nó là Class `Cherry`

```java
/*
 Đánh dấu class bằng @Component
 Class này sẽ được Spring Boot hiểu là một Bean (hoặc dependency)
 Và sẽ được Spring Boot quản lý
*/
@Component
public class Cherry implements Keycap {

    @Override
    public void insert() {
        System.out.println("Gắn key cap cherry profile vào keyboard");
    }
}
```

Và chạy chương trình
```java
@SpringBootApplication
public class App {
   public static void main(String[] args) {
      // ApplicationContext chính là container, chứa toàn bộ các Bean
      ApplicationContext context = SpringApplication.run(App.class, args);

      // Khi chạy xong, lúc này context sẽ chứa các Bean có đánh
      // dấu @Component.

      // Lấy Bean ra bằng cách
     	Keycap minila = context.getBean(Cherry.class);

      // In ra để xem thử nó là gì
      System.out.println("Instance: " + minila);
      // xài hàm wear()
      minila.insert();
   }
}
```

Bạn sẽ thấy `Keycap` lúc này chính là `Cherry`. Class đã được đánh dấu là `@Component`.

**Spring Boot** khi chạy sẽ dò tìm toàn bộ các *Class* cùng cấp hoặc ở trong các *package* thấp hơn so với class `BlogApplication` mà bạn cung cấp cho Spring (Chúng ta có thể cấu hình việc tìm kiếm này, sẽ đề cập sau). 
Trong quá trình dò tìm này, khi gặp một *class* được đánh dấu `@Component` thì nó sẽ tạo ra một *instance* và đưa vào `ApplicationContext` để quản lý.

### @Autowired <a name="spring_autowired"></a>

Bây giờ mình tạo ra một Class `MyKeyboard` và có một thuộc tính là `Keycap`.

Mình cũng đánh dấu `MyKeyboard` là một `@Component`. Tức **Spring Boot** cần tạo ra một *instance* của `MyKeyboard` để quản lý.

```java
@Component
public class MyKeyboard {
    @Autowired
    Keycap keycap;

    public MyKeyboard(Keycap keycap) {
        this.keycap = keycap;
    }
}
```

Tôi đánh dấu thuộc tính `Keycap` của `MyKeyboard` bởi Annotation `@Autowired`. Điều này nói với **Spring Boot** hãy tự *inject (tiêm)* một instance của `Keycap` vào thuộc tính này khi khởi tạo `MyKeyboard`.

Và chạy chương trình

```java
@SpringBootApplication
public class App {
   public static void main(String[] args) {
      // ApplicationContext chính là container, chứa toàn bộ các Bean
      ApplicationContext context = SpringApplication.run(App.class, args);

      // Khi chạy xong, lúc này context sẽ chứa các Bean có đánh
      // dấu @Component.

      // Lấy Bean ra bằng cách
      Keycap cherry = context.getBean(Cherry.class)

      // In ra để xem thử nó là gì
      System.out.println("Output Instance: " + cherry);
      // xài hàm wear()
      cherry.insert();

      MyKeyboard gk61 = context.getBean(MyKeyboard.class);

      System.out.println("Keyboard Instance: " + gk61);

      System.out.println("Keyboard Keycap: " + gk61.keycap);

      gk61.keycap.insert();
   }
}
```

**Spring Boot** đã tự tạo ra một `MyKeyboard` và trong quá trình tạo ra đó, nó truyền `Keycap` vào làm thuộc tính.

### Singleton

Điều đặc biệt là các `Bean` được quản lý bên trong `ApplicationContext` đều là ***singleton***. Bạn chắc đã để ý điều này từ các *Output* ở phía trên.

`Keycap` ở 2 đối tượng trên là một.

Tất cả những `Bean` được quản lý trong `ApplicationContext` đều chỉ được tạo ra **một lần duy nhất** và khi có `Class` yêu cầu `@Autowired` thì nó sẽ lấy đối tượng có sẵn trong `ApplicationContext` để inject vào.

Trong trường hợp bạn muốn mỗi lần sử dụng là một instance hoàn toàn mới. Thì hãy đánh dấu `@Component` đó bằng `@Scope("prototype")`

```java
@Component
@Scope("prototype")
public class Cherry implements Keycap {
   @Override
   public void wear() {
      System.out.println("Gắn key cap cherry profile vào keyboard");
   }
}
```

### Annotation Primary <a name="spring_primary"></a>

Trong ví dụ trên nếu ta có thêm nhiều class tương tự class `Cherry` (implement `Keycap`) thì chương trình sẽ báo lỗi 

```
Parameter 0 of constructor in com.example.demo.MyKeyboard required a single bean
```

Để fix chúng ta cần đặt một trong những class implement `Keycap` là `@Primary`.

## Spring Bean Life Cycle, @PostConstruct và @PreDestroy <a name="springboot_2"></a>

### @PostConstruct <a name="spring_postconstruct"></a>

`@PostConstruct` được đánh dấu trên một method duy nhất bên trong `Bean`. `IoC Container` hoặc `ApplicationContext` sẽ gọi hàm này **sau khi** một `Bean` được tạo ra và quản lý.

```java
@Component
public class Girl {

   @PostConstruct
   public void postConstruct(){
      System.out.println("\t>> Đối tượng MyKeyboard sau khi khởi tạo xong sẽ chạy hàm này");
   }
}
```

### @PreDestroy <a name="spring_predestroy"></a>

`@PreDestroy` được đánh dấu trên một method duy nhất bên trong `Bean`. `IoC Container` hoặc `ApplicationContext` sẽ gọi hàm này **trước khi** một `Bean` bị xóa hoặc không được quản lý nữa.

```java
@Component
public class Girl {

   @PreDestroy
   public void preDestroy(){
      System.out.println("\t>> Đối tượng MyKeyboard trước khi bị destroy thì chạy hàm này");
   }
}
```

### Bean Life Cycle <a name="spring_beanlifecycle"></a>

**Spring Boot** từ thời điểm chạy lần đầu tới khi *shutdown* thì các `Bean` nó quản lý sẽ có một vòng đời được biểu diễn như ảnh dưới đây:

![](./img/spring-bean-life-cycle.jpg)

Cần hiểu như sau:

1. Khi `IoC Container (ApplicationContext)` tìm thấy 1 `Bean` cần quản lý, nó sẽ khởi tạo bằng `constructor`

2. *Inject dependencies* vào `Bean` bằng Setter, và thực hiện các quá trình cài đặt khác vào `Bean` như `setBeanName`, `setBeanClassLoader`, v.v..

3. Gọi hàm `PostConstruct`

4. Tiền xử lý sau khi `PostConstruct` được gọi

5. `Bean` sẵn sàng để hoạt động

6. Nếu `IoC Container` không quản lý `Bean` nữa hoặc bị shutdown nó sẽ gọi hàm `@PreDestroy` trong `Bean`

7. Xóa `Bean`

### Ví dụ

Thêm `@PostConstruct` và `@PreDestroy` vào class `Girl` trong chương trước.

```java
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class MyKeyboard {

   @PostConstruct
   public void postConstruct(){
      System.out.println("\t>> Đối tượng MyKeyboard sau khi khởi tạo xong sẽ chạy hàm này");
   }

   @PreDestroy
   public void preDestroy(){
      System.out.println("\t>> Đối tượng MyKeyboard trước khi bị destroy thì chạy hàm này");
   }
}
```

và kết quả 

```
> Trước khi IoC Container được khởi tạo
> Trước khi IoC Container được khởi tạo
	>> Đối tượng MyKeyboard sau khi khởi tạo xong sẽ chạy hàm này
> Sau khi IoC Container được khởi tạo
> Trước khi IoC Container destroy MyKeyboard
	>> Đối tượng MyKeyboard trước khi bị destroy thì chạy hàm này
> Sau khi IoC Container destroy MyKeyboard
```

Bạn sẽ thấy dòng *"Trước khi IoC Container được khởi tạo"* được chạy 2 lần.

> Điều này xảy ra bởi vì hàm `App.main(args)` được chạy 2 lần!

Lần đầu là do chúng ta chạy.

Lần thứ hai là do **Spring Boot** chạy sau khi nó được gọi `SpringApplication.run(App.class, args)`. Đây là lúc mà **IoC Container** (`ApplicationContext`) được tạo ra và đi tìm `Bean`.

## @Component vs @Service vs @Repository <a name="springboot_3"></a>

### Kiến trúc trong spring boot <a name="springboot_architecture"></a>

Kiến trúc MVC trong Spring Boot được xây dựng dựa trên tư tưởng "độc lập" kết hợp với các nguyên lý thiết kế hướng đối tượng (một đại diện tiêu biểu là Dependency Inversion). 

> Độc lập ở đây ám chỉ việc các layer phục vụ các mục đích nhất định, khi muốn thực hiện một công việc ngoài phạm vi thì sẽ đưa công việc xuống các layer thấp hơn.

Kiến trúc Controller-Service - Repository chia project thành 3 lớp:

![](./img/controller-service-repository.png)

- **Consumer Layer hay Controller**: là tầng giao tiếp với bên ngoài và handler các request từ bên ngoài tới hệ thống.

- **Service Layer**: Thực hiện các nghiệp vụ và xử lý logic

- **Repository Layer**: Chịu trách nhiệm giao tiếp với các DB, thiết bị lưu trữ, xử lý query và trả về các kiểu dữ liệu mà tầng Service yêu cầu.

### @Controller vs @Service vs @Repository <a name="springboot_csr"></a>

Để phục vụ cho kiến trúc ở trên, **Spring Boot** tạo ra 3 Annotation là `@Controller`,  `@Service` và `@Repository` để chúng ta có thể đánh dấu các tầng với nhau.

- `@Service` Đánh dấu một Class là tầng `Service`, phục vụ các logic nghiệp vụ.

- `@Repository` Đánh dấu một Class Là tầng `Repository`, phục vụ truy xuất dữ liệu.

### Ví dụ

Đầu tiên ta tạo 1 class `Girl` làm `model`

```java
public class Girl {
   private String name;

   public Girl(String name) {
      this.name = name;
   }

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }

   @Override
   public String toString() {
      return "Girl(" + this.name + ")";
   }
}
```

Tạo ra một interface `GirlRepository` để giao tiếp với DB.

```java
public interface GirlRepository {
   /**
   * Tìm kiếm một cô gái trong database theo tên
   * @param name
   * @return
   */
   Girl getGirlByName(String name);
}
```

Kế thừa `GirlRepository` và đánh dấu nó là `@Repository`

```java
@Repository
public class GirlRepositoryImpl implements GirlRepository {

   @Override
   public Girl getGirlByName(String name) {
      // Ở đây tôi ví dụ là database đã trả về
      // một cô gái với tên đúng như tham số

      // Còn thực tế phải query trong csđl nhé.
      return new Girl(name);
   }
}
```

Tạo ra một class `GỉrlService` để giải quyết các logic nghiệp vụ. Lớp `GirlService` sẽ giao tiếp với DB thông qua `GirlRepository`.

```java
import org.apache.commons.lang3.RandomStringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class GirlService {
   @Autowired
   private GirlRepository girlRepository;

   public Girl getRandomGirl(){
      // Random 1 cái tên độ dài 10
      String name = randomGirlName(10);

      // Gọi xuông tầng repository để query lấy một cô gái tên là "name" trong database
      return girlRepository.getGirlByName(name);
   }

   public String randomGirlName(int length) {
      // Random một string có độ dài quy định
      // Sử dụng thư viện Apache Common Lang
      return RandomStringUtils.randomAlphanumeric(length).toLowerCase();
   }
}
```

và 

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class App {

   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(App.class, args);

      // Lấy ra bean GirlService
      GirlService girlService = context.getBean(GirlService.class);
      // Lấu ra random một cô gái từ tầng service
      Girl girl = girlService.getRandomGirl();
      // In ra màn hình
      System.out.println(girl);

   }
}
```

Về bản chất `@Service` và `@Repository` cũng chính là `@Component`. Nhưng đặt tên khác nhau để giúp chúng ta phân biệt các tầng với nhau.

Trong các bài đầu tiên chúng ta đã biết `@Component` đánh dấu cho **Spring Boot** biết Class đó là `Bean`. Và hiển nhiên `@Service` và `@Repository` cũng vậy. Vì thế ở ví dụ trên chúng ta có thể lấy `GirlService` từ `ApplicationContext`.

Về bản chất thì bạn có thể sử dụng thay thế 3 Annotation `@Component`, `@Service` và `@Repository` cho nhau mà không ảnh hưởng gì tới code của bạn cả. Nó vẫn sẽ hoạt động.

Tuy nhiên từ góc độ thiết kế thì chúng ta cần *phân rõ* 3 Annotation này cho các Class đảm nhiệm đúng nhiệm vụ của nó.

- `@Service` gắn cho các Bean đảm nhiệm *xử lý logic*
- `@Repository` gắn cho các Bean đảm nhiệm *giao tiếp với DB*
- `@Component` gắn cho các Bean khác.

## Component Scan <a name="springboot_componentscan></a>

Là cách **Spring Boot** tìm kiếm `Bean` trong project.

**Spring Boot** khi chạy sẽ dò tìm toàn bộ các `Class` cùng cấp hoặc ở trong các `package` thấp hơn và tạo ra `Bean` từ các `Class` tìm thấy.

Có 2 cách để tùy chỉnh cấu hình **Spring Boot** chỉ tìm kiếm các `Bean` trong một package nhất định:

1. `@ComponentScan`.

2. Sử dụng `scanBasePackages` tromg `@SpringBootApplication`.

### Sử dụng `@ComponentScan`

```java
@ComponentScan("com.example.blog.other")
@SpringBootApplication
public class App {
   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(App.class, args);
      try {
         Girl girl = context.getBean(Girl.class);
         System.out.println("Bean: " + girl.toString());
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }

      try {
         OtherGirl otherGirl = context.getBean(OtherGirl.class);
         if (otherGirl != null) {
               System.out.println("Bean: " + otherGirl.toString());
         }
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }
   }
}
```

### Sử dụng `scanBasePackages`

```java
@SpringBootApplication(scanBasePackages = "com.example.blog.other")
public class App {
   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(App.class, args);
      try {
         Girl girl = context.getBean(Girl.class);
         System.out.println("Bean: " + girl.toString());
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }

      try {
         OtherGirl otherGirl = context.getBean(OtherGirl.class);
         if (otherGirl != null) {
               System.out.println("Bean: " + otherGirl.toString());
         }
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }
   }
}
```

Theo như 2 ví dụ này, ta chỉ tìm các `Bean` trong `com.example.blog.other` nên kết quả sẽ như thế này:

```
Bean Girl không tồn tại
Bean: OtherGirl.java
```

> Có thể cấu hình để tìm `Bean` ở nhiều package khác nhau 

```java
@ComponentScan({"com.example.blog","com.example.blog.other"})
// hoặc
@SpringBootApplication(scanBasePackages = {"com.example.blog", "com.example.blog.other"})
```

## Spring Boot @Configuration và @Bean <a name="springboot_conf_bean"></a>

`@Configuration` là một Annotation đánh dấu trên một `Class` cho phép **Spring Boot** biết được *đây là nơi định nghĩa ra các `Bean`*.

`@Bean` là một Annotation được đánh dấu trên các method cho phép **Spring Boot** biết được *đây là `Bean`* và sẽ *thực hiện đưa `Bean` này vào `Context`*.

`@Bean` sẽ nằm trong các class có đánh dấu `@Configuration`.

### Ví dụ

viết 1 class `SimpleBean`

```java
public class SimpleBean {
   private String username;

   public SimpleBean(String username) {
      setUsername(username);
   }

   @Override
   public String toString() {
      return "This is a simple bean, name: " + username;
   }

   public String getUsername() {
      return username;
   }

   public void setUsername(String username) {
      this.username = username;
   }
}
```

class `AppConfig`

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

   @Bean
   SimpleBean simpleBeanConfigure(){
      // Khởi tạo một instance của SimpleBean và trả ra ngoài
      return new SimpleBean("loda");
   }
}
```

và xử lý trong `BlogAppication`

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class BlogApplication {
   public static void main(String[] args) {

      ApplicationContext context = SpringApplication.run(BlogApplication.class, args);
      // Lấy ra bean SimpleBean trong Context
      SimpleBean simpleBean = context.getBean(SimpleBean.class);
      // In ra màn hình
      System.out.println("Simple Bean Example: " + simpleBean.toString());
   }
}
```

`SimpleBean` là một object được quản lý trong `Context` của **Spring Boot**, mặc dù trong bài này, chúng ta không hề sử dụng tới các khái niệm `@Component`.

Đằng sau chương trình, **Spring Boot** lần đầu khởi chạy, ngoài việc đi tìm các `@Component` thì nó còn làm một nhiệm vụ nữa là tìm các class `@Configuration`.

1. Đi tìm class có đánh dấu `@Configuration`
2. Tạo ra đối tượng từ class có đánh dấu `@Configuration`
3. Tìm các method có đánh dấu `@Bean` trong đối tượng vừa tạo
4. Thực hiện gọi các method có đánh dấu `@Bean` để lấy ra các `Bean` và đưa vào `Context`.

Ngoài ra, về bản chất, `@Configuration` cũng là `@Component`. Nó chỉ khác ở ý nghĩa sử dụng. (Giống với việc class được đánh dấu `@Service` chỉ nên phục vụ logic vậy).

> nếu một `Bean` có quá nhiều logic để khởi tạo và cấu hình, thì chúng ta sẽ sử dụng `@Configuration` và `@Bean` để tự tay tạo ra Bean. 

### Ví dụ cấu hình kết nối database sử dụng `@Configuration` và `@Bean`

Đầu tiên, ta tạo 1 abstract class là `DatabaseConnector` để có thể phục vụ cho nhiều ngữ cảnh

```java
public abstract class DatabaseConnector {
   private String url;

   public abstract void connect();

   public String getUrl() {
      return url;
   }

   public void setUrl(String url) {
      this.url = url;
   }
}
```

Kế thừa class này có 2 class: MySqlConnector, MongoDBConnector.

```java
public class MySqlConnector extends DatabaseConnector{
   @Override
   public void connect(){
      System.out.println("Đã kết nối tới Mysql: " + getUrl());
   }
}
```

và

```java
public class MongoDB extends DatabaseConnector{
    @Override
    public void connect(){
        System.out.println("Đã kết nối tới MongoDB: " + getUrl());
    }
}
```

Sao đó, ta tạo `Bean` trong `AppConf`

```java
@Configuration
public class AppConf {
    @Bean("mysqlConnector")
    DatabaseConnector mysqlConfigure(){
        DatabaseConnector mysqlConnector = new MySqlConnector();
        mysqlConnector.setUrl("jdbc:mysql://host1:33060/blog");
        return mysqlConnector;
    }

    @Bean("mongodbConnector")
    DatabaseConnector mongodbConfigure(){
        DatabaseConnector mongodbConnector = new MongoDB();
        mongodbConnector.setUrl("mongodb://mongodb0.example.com:27017/blog");
        return mongodbConnector;
    }
}
```

Và chạy thử,

```java
@SpringBootApplication
public class BlogApplication {

	public static void main(String[] args) {

		ApplicationContext context = SpringApplication.run(BlogApplication.class, args);

		DatabaseConnector mysql = context.getBean(MySqlConnector.class);
		mysql.connect();

		DatabaseConnector mongodb = context.getBean(MongoDB.class);
		mongodb.connect();
	}
}
```

> Thực tế, việc sử dụng `@Configuration` là hết sức cần thiết, và nó đóng vai trò là **nơi cấu hình** cho toàn bộ ứng dụng. 

Một Ứng dụng sẽ có nhiều class chứa `@Configuration` và mỗi class sẽ đảm nhận cấu hình một bộ phận gì đó trong ứng dụng.

### Ví dụ 1 đoạn code cấu hình **Spring Security**

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll();
    }

    @Bean
    @Override
    public UserDetailsService userDetailsService() {
        UserDetails user =
             User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();

        return new InMemoryUserDetailsManager(user);
    }
}
```

## Spring Boot Application Config và annotation @Value <a name="springboot_8"></a>

Trong thực tế không phải lúc nào chúng ta cũng nên để mọi thứ trong code của mình. Có những thông số tốt hơn hết nên được truyền từ bên ngoài vào ứng dụng, để giúp ứng dụng của bạn dễ dàng thay đổi giữa các môi trường khác nhau.

Để phục vụ điều này, chúng ta sẽ tìm hiểu về khái niệm config ứng dụng **Spring Boot** với `application.properties`

### application.properties

Trong Spring Boot, các thông tin cấu hình mặc định được lấy từ file resources/applications.properties.

Ví dụ, bạn muốn Spring Boot chạy trên port 8081 thay vì 8080:

```
server.port = 8081
```

Hoặc bạn muốn log của chương trình chi tiết hơn. Hãy chuyển nó sang Debug bằng cách config như sau:

```
logging.level.root=DEBUG
```

Đây là cách chúng ta có thể can thiệp vào các cấu hình của ứng dụng từ bên ngoài. Cho phép thay đổi linh hoạt tùy môi trường.

### Annotation @Value

Trong trường hợp, bạn muốn tự config những giá trị của riêng mình, thì Spring Boot hỗ trợ bạn với annotation `@Value`

Ví dụ, tôi muốn cấu hình cho thông tin database của tôi từ bên ngoài ứng dụng

```
loda.mysql.url=jdbc:mysql://host1:33060/loda
```

> `@Value` được sử dụng trên thuộc tính của class, Có nhiệm vụ lấy thông tin từ file properties và gán vào biến.

```java
public class AppConfig {
    // Lấy giá trị config từ file application.properties
    @Value("${loda.mysql.url}")
    String mysqlUrl;
}
```

Thông tin truyền vào annottaion `@Value` chính là tên của cấu hình đặt trong dấu `${name}`

## Spring Boot Controller và ví dụ Hello World <a name="springboot_9"></a>

### Controller 

Để xây dựng một trang web với `Spring Boot` cần tuân thủ quy trình sau:

![](./img/quy-trinh.png)

`@Controller` là nơi tiếp nhận các thông tin request từ phía người dùng. Nó có nhiệm vụ đón nhận các yêu cầu và chuyển các yêu cầu này xuống cho tầng `@Service` xử lý logic.

### Ví dụ tạo trang web Hello World

> Ví dụ nảy có sử dụng Thymeleaf: Đây là một Template Engine hỗ trợ chúng ta tạo ra các file html để trả về thông tin cho người dùng.

Mặc định trong **Spring Boot** các file html sẽ được lưu trữ trong thư mục `resources/templates`

Đầu tiên, ta tạo một `@Controller`

```java
@Controller
public class WebController {
    // Đón nhận request GET
    @GetMapping("/") // Nếu người dùng request tới địa chỉ "/"
    public String index() {
        return "index"; // Trả về file index.html
    }
}
```

và một file html đơn giản 

```html
<h1>Hello, World</h1>
```

> Đừng quên thêm thymeleaf vào file `build.gradle`

Bản thân `@Controller` cũng là một Component nên nó sẽ được **Spring Boot** quản lý. 

**Spring Boot** sẽ lắng nghe các request từ phía người dùng và tùy theo `path` là gì, nó sẽ mapping tới vị trí hàm xử lý tương ứng trong `@Controller`.

Như ví dụ trên, tôi sử dụng `GET` vào địa chỉ `localhost:8080/` ( đường dẫn là /). **Spring Boot** sẽ gọi tới hàm có gắn `@GetMapping("/")` và yêu cầu hàm này xử lý `request` này.

### Ví dụ mở rộng

Ví dụ nhập tên vào và xuất ra Hello, tên.

Đầu tiên, tạo các file html làm giao diện cho web

*index.html*

```HTML
<head>
    <title>Hello World</title>
</head>
<body>

<h1>Demo website Spring Boot</h1>

<a href="/about">About</a>

<form method="get" action="/hello">
    <input type="input" name="name">
    <button type="submit">Submit</button>
</form>
</body>
```

*about.html*

```HTML
<head>
    <meta charset="UTF-8">
    <title>About</title>
</head>
<body>
    <h1>Tôi là tôi</h1>
</body>
```

*hello.html*

```html
<head>
    <title>Hello World</title>
</head>
<body>
    <h1 th:text="'Hello, ' + ${name}"></h1>
    <a href="/">Trang chủ</a>
</body>
```

Sau đó, mapping các `path` với các hàm xử lý trong `WebController`

```java
@Controller
public class WebController {

    @GetMapping("/")
    public String index(){
        return "index";
    }

    @GetMapping("/about")
    public String about(){
        return "about";
    }

    @GetMapping("/hello")
    // Model là một object của Spring Boot, được gắn vào trong mọi request.
    public String hello(@RequestParam(name = "name", required = false, defaultValue = "World") String name, Model model){
        // Gắn vào model giá trị name nhận được
        model.addAttribute("name", name);
        return "hello";
        // trả về file hello.html cùng với thông tin trong object Model
    }
}
```

Khi nhập tên vào form và bấm submit thì đường dẫn sẽ thành: http://localhost:8080/hello?name=vae

Ta sẽ nhận được giá trị của `name` và gán nó vào `Model`.

![](./img/example-hello-world.png)

`Model` ở đây là một object được **Spring Boot** đính kèm trong mỗi response.

`Model` chứa các thông tin mà bạn muốn trả về và **Template Engine** sẽ trích xuất thông tin này ra thành html và đưa cho người dùng.

Trong file `hello.html` tôi lấy giá trị của `name` trong `Model` ra bằng cách sử dụng cú pháp của `Thymeleaf`

```java
<h1 th:text="'Hello, ' + ${name}"></h1>
```

## Giải thích cách Thymeleaf vận hành + Expression + Demo Full <a name="springboot_10></a>

### Giới thiệu cơ bản về Thymeleaf 

**Thymeleaf** là một `Java Template Engine`. Có nhiệm vụ *xử lý và generate* ra các file HTML, XML, v.v..

Các file HMTL do **Thymeleaf** tạo ra là nhờ kết hợp dữ liệu và template + quy tắc để sinh ra một file HTML chứa đầy đủ thông tin.

Việc của bạn là cung cấp dữ liệu và quy định template như nào, còn việc dùng các thông tin đó để render ra HTML sẽ do **Thymeleaf** giải quyết.

### Cú pháp của Thymeleaf 

Cú pháp của **Thymeleaf** sẽ là một `attributes` (Thuộc tính) của thẻ HTML và bắt đầu bằng chữ `th:`.

Với cách tiếp cận này, bạn sẽ chỉ cần sử dụng các thẻ HTML cơ bản đã biết mà không cần bổ sung thêm syntax hay thẻ mới như JSP truyền thống.

Ví dụ: Để truyền dữ liệu từ biến `name` trong Java vào một thẻ `H1` của HTML.

```html
<h1 th:text="${name}"></h1>
```

Chúng ta viết thẻ `H1` như bình thường, nhưng không chứa bất cứ text nào trong thẻ. Mà sử dụng cú pháp `th:text="${name}"` để **Thymeleaf** lấy thông tin từ biến `name` và đưa vào thẻ `H1`.

Kết quả khi render ra:
Giả sử String name = "Vae"

```html
<h1>vae</h1>
```

thuộc tính `th:text` biến mất và giá trị biến `name` được đưa vào trong thẻ `H1`.

Đó là cách **Thymeleaf** hoạt động.

### Model & View trong Spring Boot

`Model` là đối tượng lưu giữ thông tin và được sử dụng bởi `Template Engine` để generate ra webpage. Có thể hiểu nó là `context` của **Thymeleaf**.

`Model` lưu giữ thông tin dưới dạng key-value

Trong template **thymeleaf**, để lấy các thông tin trong `Model`. bạn sẽ sử dụng `Thymeleaf Standard Expression`.

- `${...}`: giá trị của 1 biến (variable expression)
- `*{...}`: giá trị của 1 biến được chỉ định (variable expression on selection)
- `#{...}`: lấy message (message expression)
- `@{...}`: Lấy đường dẫn URL dựa theo context của server (URL expression)

#### Ví dụ minh họa cho từng expression

1. ${...} - variable expression

Đưa vào model một giá trị

```java
model.addAttribute("today", "Monday");
```

Để lấy giá trị của biến today, ta sử dụng `${...}`

```html
<p>Today: <span th:text="${today}"></span><p>
```
Đoạn expression trên tương ứng với:

```java
ctx.getVariable("today");
```

2. *{...} - variable expression on selections

Dấu `*` còn được gọi là asterisk syntax. Chức năng của nó giống với variable expression.

Điểm khác biệt là nó sẽ lấy giá trị của 1 biến được cho trước bởi `th:object`.

```html
<div th:object="${session.user}">
<!-- th:object tồn tại trong phạm vi của thẻ div này -->
    <p>Name: <span th:text="*{firstName}"></span></p>

    <p>Age: <span th:text="*{age}"></span></p>
</div>
```

Còn variable expression sẽ lấy giá trị trong `Model` hay `Context`

Vậy đoạn code trên tương đương với các sử dụng variable expression

```html
<div>
    <p>Name: <span th:text="${session.user.firstName}"></span></p>

    <p>Age: <span th:text="${session.user.age}"></span></p>
</div>
```

3. #{...} - message expression

Ví dụ trong file config `.properties` có một messenger chào người dùng.

```
home.welcome = Hello bà con
```

thì cách lấy nó ra nhanh nhất là:

```html
<p th:utext="#{home.welcome}">xin chào các bạn</p>
```

Đoạn text "xin chào các bạn" sẽ bị thay thế bới thymeleaf khi render `#{home.welcome}`

4. @{...} - URL expression

`@{...}` xử lý và trả ra giá trị URL theo context của máy chủ cho chúng ta.

Nếu bắt dầu bằng dấu / thì nó sẽ là Relative URL và sẽ tương ứng theo context của máy chủ của bạn.

### Demo minh họa

Tạo cấu trúc thư mục

```
src
|-  main
       |- java
          |- com.example.demo
             DemoApplication.java
             Info.java
             WebController.java
       |- resources 
          |- i18n
             |- messages.properties
             |- messages_vi.properties
             |- messages_en.properties
          |- static
             |- css
             |- js
          |- templates
          application.properties
```


nội dung file `application.properties` để cấu hình ứng dụng

```
#Chạy ứng dụng trên port 8085
server.port=8085

# Bỏ tính năng cache của thymeleaf để lập trình cho nhanh
spring.thymeleaf.cache=false

# Các message tĩnh sẽ được lưu tại thư mục i18n
spring.messages.basename=i18n/messages


# Bỏ properties này đi khi deploy
# Nó có tác dụng cố định ngôn ngữ hiện tại chỉ là Tiếng Việt
spring.mvc.locale-resolver=fixed

# Mặc định ngôn ngữ là tiếng việt
spring.mvc.locale=vi_VN
# Đổi thành tiếng anh bằng cách bỏ comment ở dứoi
# spring.mvc.locale=en_US
```

Tôi làm 1 câu chào đơn giản cho 2 vùng US và VN

*i18n/messages_vi.properties*

```
demo.hello = Chào bà con 
```

*i18n/messages_en.properties*

```
demo.hello = Hello everybody
```

Làm web thì không thể thiếu `css` và `javascript`. Các file này sẽ được lưu trử tại thư mục `resources/static`.

File `.html` là dạng template sử dụng để render ra webpage và trả về cho người dùng. Nó sẽ được lưu tại thư mục `resources/templates`

**Thymleaf** sẽ tự biết tìm đường tới những tài nguyên này.

`index.html` sẽ là file mặc định mà **Thymeleaf** tìm đầu tiên và trả về mỗi khi người dùng vào địa chỉ `/` hay `https://localhost:8085/` mà chúng ta không cần `config` gì cả.

Trong `index.html` tôi sẽ:

1. Gọi ra `bootstrap.css` và `bootstrap.js` trong thư mục `resource/static` bằng expression `@{...}`

2. Hiển thị dòng chữ chào `demo.hello` trong thư mục `resource/i18n` bằng expression `#{...}`

Tẹo template `index.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Hello World</title>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8"/>

    <!-- css -->
    <link th:href="@{/css/bootstrap.css}" rel="stylesheet">

    <!-- js -->
    <script th:src="@{/js/bootstrap.js}"></script>
</head>
    <body>
        <h1 th:utext="#{demo.hello}"></h1>

        <a th:href="@{/profile}" class="btn btn-primary">click me</a>
    </body>
</html>
```



