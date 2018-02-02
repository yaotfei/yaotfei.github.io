---
title: Spring学习笔记（三）：面向切面的Spring
date: 2017-07-23 20:50:10
categories: Spring学习笔记
tags: SpringAOP
description: 面向切面编程（AOP）在Spring框架中被作为核心组成部分之一，Spring将AOP发挥到很强大的功能。可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。
---
# 前言 #
---
一个成熟的软件系统包括许多功能，比如日志，安全和事务管理器等，这些功能需要用到应用程序的多个地方，但是我们又不想在每个点都明确调用它们。如果让应用对象只关注于自己所针对的业务，而其他的问题由其他的应用对象来处理，这会不会更好呢？  
在软件开发中，散布于应用中多处的功能被称为横切关注点，这些横切关注点从概念上是与应用的业务逻辑相分离的。把这些横切关注点与业务逻辑相分离正是面向切面编程（AOP)所要解决的问题。

# 正文 #
---
## 什么是面向切面编程 ##
在软件开发过程中，如果要重用通用功能的话，最常见的面向对象技术是继承或者委托。但是，如果在整个应用中都使用相同的基类，继承往往会导致一个脆弱的对象体系，而使用委托可能需要对委托对象进行复杂的调用。  
切面提供了取代继承和委托的另一种可选方案，而且在很多场景下更清晰简洁。在使用面向切面编程时，我们仍然在一个地方定义通用功能，但是可以通过声明的方式定义这个功能要以何种方式在何处应用，而无需修改受影响的类。横切关注点可以被模块化为特殊的类，这些类被称为切面。这样做有两个好处：首先，现在每个关注点都集中于一个地方，而不是分散到多处代码中；其次，服务模块更简洁，因为它们只包含主要的关注点的代码，而次要关注点的代码被转移到切面中了。

### AOP术语　###

AOP已经形成了自己的术语。描述切面的常用术语有通知（advice）、切点（pointcut）、和连接点（join point）。  

**通知（Advice）**    

在上面我们介绍了什么是切面————需要通用的代码被模块化为特殊的类。  
类似地，切面也有目标：它必须要完成的工作。在AOP术语中，切面的工作被称为通知。  
通知定义了切面是什么以及何时使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。它应该应用在某个方法被调用之前？之后？之前和之后都调用？还是只在方法抛出异常时调用？  
Spring切面可以应用5种类型的通知：  
> - 前置通知（Before）：在目标方法被调用之前调用通知功能；
> - 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
> - 返回通知（After-returning）：在目标方法成功执行之后调用通知；
> - 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
> - 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和之后执行自定义的行为。  

**连接点（Join point）**

我们的应用可能有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

**切点（Point）**  

一个切面不需要通知应用的所有连接点，而切点有助于缩小切面通知的连接点的范围。  
如果说通知定义了切面的“什么”和“何时”，那么切点就定义了“何处”。切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。有些AOP框架允许我们创建动态的切点，可以根据运行时的决策（比如方法的参数值）来决定是否应用通知。  

**切面（Aspect）**  

切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。  

**引入（Introduction）**    

引入允许我们向现有的类添加新方法或属性。例如，我们可以创建一个Auditable通知类，该类记录了对象最后一次修改时的状态。这很简单，只需一个方法，setLastModified(Date)，和一个实例变量来保存这个状态。然后，这个新方法和实例变量就可以被引入到现有的类中，从而可以在无需修改这些现有的类的情况下，让它们具有新的行为和状态。  

**织入（Weaving）**   

织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入：  

> - 编译器：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
> - 类加载器：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-timeweaving，LTW）就支持以这种方式织入切面。
> - 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。  

我们已经了解了一些基础的AOP术语，现在让我们再看看这些AOP的核心概念是如何在Spring中实现的。

### Spring对AOP的支持 ###  
并不是所有的AOP框架都是相同的，它们在连接点模型上可能有强弱之分。但是无论如何，创建切点来定义切面所织入的连接点是AOP框架的基本功能。  
Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于方法拦截。  

## 通过切点来选择连接点 ##
切点用于准确定位应该在什么地方应用切面的通知。通知和切点是切面的最基本元素。因此，了解如何编写切点非常重要。  
在Spring AOP中，要使用AspectJ的切点表达式语言来定义切点。关于Spring AOP的AspectJ切点，最重要的一点就是Spring仅支持AspectJ切点指示器的一个子集。下表列出了Spring AOP所支持的AspectJ 切点表达式语言来定义Spring切面：

{% asset_img pic_01.png Spring借助AspectJ的切点表达式语言来定义Spring切面%}

在Spring中尝试使用AspectJ其他指示器时，将会抛出IllegalArgument-Exception异常。  
在上面所展示的这些Spring支持的指示器时，只有execution指示器时实际执行匹配的，其他的指示器都是用来限制匹配的。这说明execution指示器是我们在编写切点定义时最主要使用的指示器。在此基础上，我们使用其他指示器来限制所匹配的切点。  

### 编写切点 ###

为了阐述Spring中的切面，我们定义一个Performance接口：

	package concert;

	public interface Performance{
		public void perform();
	}

Performance可以代表任何类型的现场表演。假设我们编写Performance的perform()方法触发的通知：

{% asset_img pic_02.png 使用AspectJ切点表达式来选择Performance的perform()方法%}

上图展现了一个切点表达式，这个表达式能够设置当perform()方法执行时触发通知的调用。  
我们使用execution()指示器选择Performance的perform()方法。方法表达式以“*”号开始，表明了我们不关心方法返回值类型。然后，我们指定了全限定类名和方法名。对于方法参数列表，我们使用两个点号（..）表明切点要选择任意的perform()方法，无论该方法的入参是什么。  
现在假设我们需要配置的切点仅匹配concert包，可以使用winthin()指示器来限制匹配：  

{% asset_img pic_03.png 使用within()指示器限制切点范围%}

我们使用了“&&”操作符把execution()和winthin()指示器连接在一起形成与（and）关系。类似地，我们可以使用“||”操作符来标识或（or）关系，而使用“!”操作符来标识非（not）操作。  
因为“&”在XML中有特殊含义，所以在Spring的XML配置里面描述切点时，我们可以使用and来代替“&&”。同样，or和not可以分别用来代替“||”和“!”。 
 
### 在切点中选择bean ###

除了Spring支持的指示器外，Spring还引入了一个新的bean()指示器，它允许我们我们在切点表达式中使用bean的ID来标识bean。bean()使用bean ID或则bean名称作为参数来限制切点只匹配特定的bean：

	execution（* concert。Performance.perform()) and bean('woodstock')

在这里，我们希望在执行Performance的perform()方法时应用通知，但限定bean的ID为woodstock。  
在某些场景下，限定切点为指定的bean获取很有意义，但我们还可以使用非操作为除了特定ID以外的其他bean应用通知：

	execution（* concert.Performance.perform()) and !bean('woodstock')

在此场景下，切面的通知会被编织到所有ID不为woodstock的bean中。

## 使用注解创建切面 ##

使用注解来创建切面是AspectJ 5所引入的关键特性。AspectJ面向注解的模型可以非常简便地通过少量注解把任意类转换成切面。  
我们已经定义了Performance接口，它是切面中切点的目标对象。现在，我们使用AspectJ注解来定义切面。  

#### 定义切面 ####

从演出的角度来看，观众是非常重要的，但是对演出本身的功能来讲，它并不是核心，这是一个单独的关注点。因此，将观众定义为一个切面，并将其应用到演出上。  

	package concert
	
	@Aspect
	public class Audience{
	
		//表演之前
		@Before("execution(** concert.Performance.perform(..))")
		public void silenceCellPhones(){
			System.out.println("Silencing cell phones");
		}
	
		//表演之前
		@Before("execution(** concert.Performance.perform(..))")
		public void takeSeats(){
			System.out.println("Taking seats");
		}
	
		//表演之后
		@AfterReturning("execution(** concert.Performance.perform(..))")
		public void applause(){
			System.out.println("CLAP CLAP CLAP CLAP!!!");
		}
	
		//表演失败之后
		@AfterThrowing("execution(** concert.Performance.perform(..))")
		public void demandRefund(){
			System.out.println("Demanding a refund");
		}
	}

Audience类使用@AspectJ注解进行了标注。该注解表明Audience不仅仅是一个POJO，还是一个切面。Audience类中的方法都使用注解来定义切面的具体行为。  
Audience有四个方法，定义了一个观众在观看演出时可能会做的事情。在演出之前，观众要就坐（takeSeats()）并将手机调至静音状态（silenceCellPhones()）。如果演出很精彩的话，观众应该会鼓掌喝彩（applause()）。不过，如果演出没有达到观众预期的话，观众会要求退款（demandRefund()）。  
可以看到，这些方法都使用了通知注解来表明他们应该在什么时候调用。AspectJ提供了五个注解来定义通知：  

{% asset_img pic_04.png Spring使用AspectJ注解来声明通知方法%}

Audience使用到了前五个注解中的三个。takeSeats()和silence CellPhones()方法都用到了@Before注解，表明它们应该在演出开始之前调用。applause()方法使用了@AfterReturning注解，它会在演出成功返回后调用。demandRefund()方法上添加了@AfterThrowing注解，这表明它会在抛出异常以后执行。  
在上面的这些注解都给定了一个切点表达式作为它的值，同时这四个方法的切点表达式都是相同的。其实，它们可以设置成不同的切点表达式。  
相同的切点表达式我们重复了四遍，如果我们只定义这个切点一次，然后每次需要的时候引用它，那么这会是一个很好的方案。  
我们可以使用@Pointcut注解能够在一个@AspectJ切面内定义可重用的切点：

	package concert
	
	@Aspect
	public class Audience{

		//定义命名的切点
		@Pointcut("execution(** concert.Performance.perform(..))")
		public void performance(){}
	
		//表演之前
		@Before("performance()")
		public void silenceCellPhones(){
			System.out.println("Silencing cell phones");
		}
	
		//表演之前
		@Before("performance()")
		public void takeSeats(){
			System.out.println("Taking seats");
		}
	
		//表演之后
		@AfterReturning("performance()")
		public void applause(){
			System.out.println("CLAP CLAP CLAP CLAP!!!");
		}
	
		//表演失败之后
		@AfterThrowing("performance()")
		public void demandRefund(){
			System.out.println("Demanding a refund");
		}
	}

在Audience中，performance()方法使用了@Pointcut注解。为@Pointcut注解设置的值是一个切点表达式，就像之前在通知注解上所设置的那样。通过在performance()方法上添加@Pointcut注解，我们实际上扩展了切点表达式语言，这样就可以在任何的切点表达式中使用performance()了，如果不这样做的话，你需要在这些地方使用那个更长的切点表达式。我们现在把所有通知注解中的长表达式都替换成了performance()。  
Performance()方法的实际内容并不重要，在这里它实际上应该是空的。其实该方法本身只是一个标识，供@Pointcut注解依附。
需要注意的是，除了注解和没有实际操作的performance()方法，Audience类依然是一个POJO。我们能够像使用其他的Java类那样调用它的方法，它的方法也能够独立地进行单元测试，这与其他的Java类并没有什么区别。Audience只是一个Java类，只不过它通过注解表明会作为切面使用而已。  
像其他的java类一样，它可以装配为Spring中的bean：

	@Bean
	public Audience audience（）{
		return new Audience();
	}  

如果就此止步的话，Audience只是Spring容器中的一个bean。即便使用了AspectJ注解，但它并不会被视为切面，这些注解不会解析，也不会创建将其转换为切面的代理。  

如果你使用javaConfig的话，可以在配置类的类级别上通过使用EnableAspectj-AutoProxy注解启用自动代理功能：

	package concert;
	
	@Configuration
	@EnableAspectJAutoProxy				//启用AspectJ自动代理
	@ComponentScan
	public class ConcertConfig{
	
		@Bean
		public Audience audience(){		//声明Audience bean
			return new Audience();
		}
	}
	
假如你在Spring中要使用XML来装配bean的话，那么需要使用Spring AOP命名空间中的<aop:aspectj-autoproxy>元素：

	<context:component-scan base-package="concert" />
	<!-- 启用AspectJ自动代理 -->
	<aop:aspectj-autoproxy />
	<!-- 声明Audience bean -->
	<bean class="concert.Audience" />	

不管你是使用JavaConfig还是XML，AspectJ自动代理都会为使用@Aspect注解的bean创建一个代理，这个代理会围绕着所有该切面的切点所匹配的bean。在这种情况下，将会为Concertbean创建一个代理，Audience类中的通知方法将会在perform()调用前后执行。  
Spring的AspectJ自动代理仅仅使用@AspectJ作为创建切面的指导，切面依然是基于代理的。在本质上，它依然是Spring基于代理的切面。这一点非常重要，因为这意味着尽管使用的是@AspectJ注解，但我们仍然限于代理方法的调用。如果想利用AspectJ的所有能力，我们必须在运行时使用AspectJ并且不依赖Spring来创建基于代理的切面。  

#### 创建环绕通知 ####
环绕通知是最强大的通知类型。他能够让你所编写的逻辑将被通知的目标方法完全包装起来。实际上就像在一个通知方法中同时编写前置通知和后置通知。
我们使用环绕通知重新实现Audience切面：

	package concert
	
	@Aspect
	public class Audience{

		//定义命名的切点
		@Pointcut("execution(** concert.Performance.perform(..))")
		public void performance(){}
		//环绕通知方法
		@Around("performance()*)
		public void watchPerformance(ProceedingJoinPoint jp){
			try{
				System.out.println("Silencing cell phones");
				System.out.println("Taking seats");
				jp.proceed();
				System.out.println("CLAP CLAP CLAP!!!");
			}catch(Throwable e){
				System.out.println("Demanding a refund");
			}
		}
	}

@Around注解表明watchPerformance()方法会作为performance()切点的环绕通知。在这个通知中，观众在演出前会将手机调至静音并就坐，演出结束后会鼓掌喝彩，如果演出失败，观众会要求退款。  
在这个新的通知方法中，ProceedingJoinPoint作为参数，这个对象必须有，因为你还要在通知中通过它来调用被通知的方法。通知方法中可以做任何的事情，当要将控制权交给被通知的方法时，它需要调用ProceedingJoinPoint的proceed()方法。如果不调用这个方法，你的通知实际上会阻塞对被通知的调用。

## 在XML中声明切面 ##

在Spring开发中中我们基于这样一种原则：基于注解的配置要优于基于java的配置，基于java的配置要优于基于XML的配置。但是，如果要声明切面，但是又不能为通知类添加注解时，就必须转向XML配置了。  
在Spring的aop命名空间中，提供了许多元素用来在XML中声明切面：

{% asset_img pic_05.png Spring的AOP配置元素%}

aop命名空间的元素能够让我们直接在Spring配置声明切面，而不需要注解。  
我们重新看一下Audience类，我们将所有的AspectJ注解去掉

	package concert
	
	public class Audience{

		public void performance(){}
	
		public void silenceCellPhones(){
			System.out.println("Silencing cell phones");
		}

		public void takeSeats(){
			System.out.println("Taking seats");
		}
	
		public void applause(){
			System.out.println("CLAP CLAP CLAP CLAP!!!");
		}

		public void demandRefund(){
			System.out.println("Demanding a refund");
		}
	}

Audience类现在没有任何特别之处，就是有几个方法的简单java类。我们可以像其他类一样把它注册为Spring应用上下文中的bean。  

### 声明前置和后置通知 ###

现在我们通过XML将无注解的Audience声明为切面：

	<aop:config>
		<aop:aspect ref="audience">		<!-- 引用Audience bean -->
			<aop:before pointcut="execution(**concert.Performance.perform(..))" method="silenceCellPhones" />

			<aop:before pointcut="execution(**concert.Performance.perform(..))" method="takeSeats" />

			<aop:after-returning pointcut="execution(**concert.Performance.perform(..))" method="applause" />

			<aop:after-throwing pointcut="execution(**concert.Performance.perform(..))" method="demandRefund" />
		</aop:aspect>
	</aop:config>

在基于AspectJ注解的通知中，我们使用@pointcut注解消除了这些重复的内容，而在基于XML的切面声明中，我们需要使用<aop:pointcut>元素：

	<aop:config>
		<aop:aspect ref="audience">		<!-- 引用Audience bean -->
		<!-- 定义切点 -->
		<aop:pointcut id="performance" expression="execution(**concert.Performance.perform(..))" />
			<aop:before pointcut-ref="performance" method="silenceCellPhones" />

			<aop:before pointcut-ref="performance" method="takeSeats" />

			<aop:after-returning pointcut-ref="performance" method="applause" />

			<aop:after-throwing pointcut-ref="performance" method="demandRefund" />
		</aop:aspect>
	</aop:config>
	
现在切点是在一个地方定义的，并且被多个通知元素所引用。

### 声明环绕通知 ###

我们在使用前置通知和后置通知的时候如果不使用成员变量存储信息的话，在前置通知和后置通知之间共享信息非常麻烦。  
例如，我们希望观众确保一直关注演出，并报告每个参赛者表演了多长时间。使用前置通知和后只通知实现该功能的唯一方式是在前置通知中记录开始时间并在某个后置通知中报告表演耗费的时间。但这样的话我们必须在一个成员变量中保存开始时间。因为Audience是单例的，如果这样的话，将会存在线程安全问题。  

如果使用环绕通知，我们可以完成前置和后置通知所实现的相同功能，而且只需要在一个方法中实现。因为整个通知逻辑是在一个方法内实现的，所以不需要使用成员变量保存信息。  
在上面的Audience类的watchPerformance()方法中，去掉注解：

	package concert
	
	public class Audience{

		public void watchPerformance(ProceedingJoinPoint jp){
			try{
				System.out.println("Silencing cell phones");
				System.out.println("Taking seats");
				jp.proceed();	//执行被通知的方法
				System.out.println("CLAP CLAP CLAP!!!");
			}catch(Throwable e){
				System.out.println("Demanding a refund");
			}
		}
	}

声明环绕通知与声明其他类型的通知并没有太大的区别，我们需要做的就是使用<aop:around>元素：

	<aop:config>
		<aop:aspect ref="audience">		<!-- 引用Audience bean -->
		<!-- 定义切点 -->
		<aop:pointcut id="performance" expression="execution(**concert.Performance.perform(..))" />
			<!-- 声明环绕通知 -->
			<aop:around pointcut-ref="performance" method="watchPerformance" />
		</aop:aspect>
	</aop:config>

<aop:around>指定了一个切点和一个通知方法的名字。在这里，我们使用跟之前一样的切点，但是为该切点所设置的method属性值为watchPerformance()方法。

# 总结 #
---
AOP是面向对象编程的一个强大补充。通过AspectJ，我们现在可以把之前分散在应用各处的行为放入可重用的模块中。我们显示地声明在何处如何应用该行为。这有效减少了代码冗余，并让我们的类关注自身的主要功能。  
Spring提供了一个AOP框架，让我们把切面插入到方法执行的周围。现在我们已经学会如何把通知织入前置、后置和环绕方法的调用中，以及为处理异常增加自定义的行为。

# 参考 #
---
[Spring实战](https://book.douban.com/subject/26767354/)

