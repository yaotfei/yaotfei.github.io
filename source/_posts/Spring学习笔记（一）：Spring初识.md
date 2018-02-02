---
title: Spring学习笔记（一）：Spring初识
date: 2017-07-16 13:30:05
categories: Spring学习笔记
tags: Spring
description: 在工作中一直用Spring相关框架，但都是用到什么学什么，没有一个完整的知识框架，所以导致对Spring的认知也不完整，所以，特地买了相关资料从新认识Spring。
---
# 前言 #
---
Spring是一个开源框架，，Spring可以做非常多的事情，但归根到底支撑Spring的仅仅是少许的基本理念，所有的理念都可以追溯到Spring最根本的使命上：简化java开发。
Spring是如何简化java开发的？为了降低java开发的复杂性，Spring采取了一下4中关键策略：

>- 基于POJO的轻量级和最小侵入性的编程。
>- 通过依赖注入和面向接口实现松耦合。
>- 基于切面和惯例进行声明式编程。
>- 通过切面和模版减少样板式代码。

几乎Spring所做的任何事情都可以追溯到上述的一条或多条策略。
# 正文 #
---
## 激发POJO的潜能 ##
### 什么是POJO ###
首先，什么是POJO。
POJO（Plain Ordinary java Object）简单的java对象，实际就是普通的javaBeans，是为了。。。等等，那什么是javaBean，从大学开始就一直听说javaBean，当时就是一个简单的对象。
	{% asset_img pic_01.png pic_01.png%}
上面是百度百科的简介，总的来说就是符合一定规范的（其实也没有严格的规范）java对象。
而POJO就是普通的javaBean，使用POJO名称是为了避免和EJB混淆起来, 而且简称比较直接. 其中有一些属性及其getter setter方法的类,没有业务逻辑。
### 此bean非彼bean ###
虽然Spring用Bean或者javaBean來标识应用组件，（那什么是组件，暂且理解为为了实现Spring功能的工具吧，比如，你要修理自行车，而扳手就相当于一个组件）但并不意味着Spring组件必须要遵循javaBean规范，一个Spring组件可以是任何形式的POJO。
那问题來了，java的bean和Spring中的bean有什么区别呢？可以说是javabean的发展，但已经完全不是一回事儿了。

>- 用处不同：传统的javabean更多地作为值传递参数，而Spring中的bean用处几乎无处不在，任何组件都可以被称为bean。
>- 写法不同：传统javabean作为值对象，要求每个属性都提供getter和setter方法；但spring中的bean只需为接受设值注入的属性提供setter方法。
>- 生命周期不同：传统javabean作为值对象传递，不接受任何容器管理其生命周期；spring中的bean有spring管理其生命周期行为。

所有可以被spring容器实例化并管理的java类都可以称为bean。
SpringBean。
如果是用过其他框架会发现很多框架通过强迫应用继承他们的类或者实现他们当接口从而导致应用与框架绑死。而Spring竭力避免因自身的API而弄乱你的应用代码，不会强迫你实现Spring规范的接口或继承Spring规范的类，最多就是一个类或许会是用Spring注解，但他依旧是POJO。
举个例子：

	public classHelloWorldBean{
		public String sayHello(){
			return "Hello Wordl";
		}
	}
可以看到，这是一个简单普通的java类，没有任何地方表明它是一个Spring组件。Spring的非侵入编程模型意味着这个类在Spring应用和非Spring应用中都可以发挥同样的作用。那么Spring是如何是用的它们的呢。Spring通过DI來装配它们。
### 依赖注入 ###
任何一个有实际意义的应用都会由两个或者更多的类组成，这些类相互之间进行协作来完成特定的业务逻辑。按照传统的做法，每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将会导致高度耦合和难以测试的代码。
举个例子：
	
	public class LearnJava implements Student{

		private LearnJavaSkill learn;	//学习java的能力

		public LearnJava(){
			this.learn = new LearnJavaSkill();
		}
		
		public void toLearn(){
			learn.toLearnJava();	//学习java	
		}
	}

比如代码中一个学生想要学习java，就要有学习java的能力，于是new了个LearnJavaSkill(),这样使得 LearnJava和LearnJavaSkill耦合到了一起，这个学生java学的很好，但如果还想学C，就爱莫能助了。
耦合具有两面性，一方面紧密耦合的代码难以测试，难以复用，另一方面，一定程度的耦合也是必须的，为了完成一些功能，耦合是必须的。  
通过DI（依赖注入），对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系

	public class SmartStudent implements Student{

		private Learn learn;	//学习能力
		
		//learn被注入
		public smartStudent(Learn learn){
			this.learn = learn;
		}
		
		public void learnJava(){
			learn.toLearnJava();
		}
	}

聪明的学生都具有很强的学习能力。可以看到不同于之前的LearnJava，学生没有自行创建学习英语的能力，而是在构造的时候把学习能力作为构造器的参数传递进来，这样学生就具有了很强的学习能力了。这是依赖注入的方式之一：构造器注入。  
而传入的learn类型是Learn，也就是说所有的学习任务只要实现Learn接口，学生具有了学习java、PHP、C++的能力就可以轻松学习java，PHP或则C++了。  
这里的学生没有和特定的学习任务发生耦合，对于学生来说，学习能力很强，那么至于学什么对于学生来说都可以轻松应对。  
现在聪明的学生可以具有你传递给他的任何学习能力，但怎么才能吧某个学习能力传递给他呢，加入学生想学习java，则上面两种方法都适用，但如果学生想学PHP呢。  
	
	public class LearnPHP implements Learn{
		
		public void learnPHP(){
			log("Hello PHP");
		}
	}

创建应用组件（smartStudent和learnPHP就是Spring组件）之间协作的行为通常称为装配。Spring有多种装配bean的方式，才用XML是常见的一种。  
下面的配置文件student.xml将smartStudent和learnPHP装配到了一起：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd">

		<!-- 声明SmartStudent为bean -->
		<bean id="smartStudent class="xx.xx.SmartStudent">
    		<constructor-arg ref="learnPHP" />
		</bean>

		<!-- 声明LearnPHP为bean -->
		<bean id="learnPHP" class="xx.xx.LearnPHP">
   
		</bean>
	</beans>

Spring配置文件中<bean>标签把smartStudent和learnPHP声明为标签（bean中的id只是bean的唯一标识）。是用<constructor-arg>把另一个bean注入。
如果你不喜欢XML不符合你的习惯，Spring还支持是用java來描述配置信息。比如下面的java配置和上面的配置文件作用一致：

	@Configuration
	public class StudentConfig{
		@Bean
		public Student smartStudent(){
			return new SmartStudent(learnPHP());	
		}

		@Bean
		public Learn learnPHP(){
			log("Hello PHP");
		}
	}

不管是用XML还是java配置，DI已经声明了smartStudent和learnPHP的关系，接下来我们只需要装载XML配置文件，并把应用启动起来。  
Spring通过应用上下文（Application Context）装载bean的定义并把它们组装起来，Spring应用上下文全权负责对象的创建和组装。Spring自带了多种应用上下文的实现，它们的区别仅仅在于如何加载配置。  
因为student.xml中的bean是使用XML文件进行配置的，所以选择ClassPathCmlApplicationContext作为应用上下文相对是比较合适的（对于基于java的配置，Spring提供了AnnotationConfigApplicationContext）。该类加载位于应用程序类路径下的一个或者多个XML配置文件。

	public class SmartStudentMain {
		public static void main(String[] args) throws Exception {
			ClassPathXmlApplicationContext context = 
			new ClassPathXmlApplicationContext("META-INF/spring/student.xml");	//加载Spring上下文
    		SmartStudent smartStudent = context.getBean(smartStudent.class);	//获取smartStudent bean
    		smartStudent.learnPHP();		//使用smartStudent
    		context.close();
		}
	}

这里的main()是基于配置文件创建了Spring应用上下文。

### 应用切面 ###
DI能够让软件组件保持松散耦合，面向切面编程（AOP）允许你把遍布应用各处的功能分离出来形成可重用的组件。  
一个可用的系统由许多不同的组件组成，每一个组件各负责一块特定的功能，除了实现自身核心的功能之外，这些组件还经常承担这额外的职责，如日志，事务管理和安全这样的服务。  
如果将这些关注点分散到多个组件中去，你的代码将会带来复杂性。  
AOP能够使这些服务模块化，并以声明的方式将它们应用到它们需要影响的组件中去。所造成的结果就是这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务所带来复杂性。总之，AOP能够确保POJO的简单性。  
学生学完相关课程，老师想要检验一下学生的学习效果，就要在学生学习之前和学习后出测试题，下面的代码定义了考试的功能

	public class Teacher{

		private Exam exam;	//老师的考试功能

		//注入exam
		public Teacher(Exam exam){
			this.exam = exam;
		}

		//学习前调用
		public void examBeforeLearn(){
			exam.log("Don't pass");
		}
		
		//学习后调用
		public void examAfterLearn(){
			exam.log("pass");
		}
	}

上面的Teacher.java有两个方法，在学生学习前examBeforeLearn()被调用，在学生学习完后examAfterLearn()方法被调用，输出考试结果。  
如果想要把功能实现就要把Teacher.java放入smartStudent.java中，smartStudent就必须要调用Teacher，

	public class SmartStudent implements Student{

		private Learn learn;	//学习能力
		private Teacher teacher;
		
		//learn被注入
		public smartStudent(Learn learn){
			this.learn = learn;
		}
		
		public void learnJava(){
			teacher.examBeforeLearn();	//学习前调用
			learn.toLearnJava();
			teacher.examAfterLearn();	//学习后调用
		}
	}

剩下要做的就是在Spring配置文件中声明Teacher为bean，然后把Teacher注入到smartStudent中，这样程序就能运行起来。  
一切看起来都很完美，但是仔细想想，老师考试学生应该是老师的分内事儿，不需要学生去调用他，而且，如果老师要老师学生就必须把老师注入进来，如果有些学生不需要被老师检测，如果teacher为null呢，那我们还要判断这些超出预期的功能，那么smartStudent类就变的越来越复杂。  
如果把Teacher抽象为一个切面，无论你想不想考试，老师就在那，不离不弃。你需要做的就是在Spring中声明老师为一个切面：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd">

		<!-- 声明SmartStudent为bean -->
		<bean id="smartStudent class="xx.xx.SmartStudent">
    		<constructor-arg ref="learnPHP" />
		</bean>

		<!-- 声明LearnPHP为bean -->
		<bean id="learnPHP" class="xx.xx.LearnPHP">
   
		</bean>
		
		<!-- 声明Teacher为bean -->
		<bean id="teacher" class="xx.xx.Teacher">
    		
		</bean>

		<aop:config>
    		<aop:aspect ref="teacher">
				<!-- 定义切点 -->
      			<aop:pointcut id="toLearnJava" expression="execution(* *.toLearnJava(..))"/>	
        
			<!-- 声明前置通知 -->
      		<aop:before pointcut-ref="toLearnJava" method="examBeforeLearn"/>	

			<!-- 声明后置通知 -->
      		<aop:after pointcut-ref="toLearnJava" method="examAfterLearn"/>
    		</aop:aspect>
		</aop:config>
	</beans>

上面的配置把Teacher声明了一个切面。只需要声明在toLearnJava()方法执行前运行examBeforeLearn(),在toLearnJava()方法执行后运行examAfterLearn()。这样SmartStudent.java就和Teacher解耦了。

### 使用模版消除板式代码 ###

想象一下当我们第一次使用jdbc的时候，首先你得创建一个数据库链接，然后创建一个语句对象，最后才能进行查询，而且为了异常还要捕捉SQLException,最后还要关闭数据库链接、语句、和结果集，依然要捕捉SQLException。只有少量相关业务逻辑代码。其他的代码都是jdbc的样式模版。  
Spring通过模版封装来消除板式代码。Spring的jdbcTemplate使得执行数据库操作时，避免了传统的jdbc样式代码，把更多的尽力投入到自己的业务代码中。

# 总结 #
---
上面只是简单介绍的Spring的个别功能，而且还有更好的办法实现上面的功能，比如注解。这些只是Spring很少的一部分功能，其他的比如Spring web端的Spring MVC，持久层的Spring Data，简化Spring开发的Spring Boot等。
	
# 参考 #
---
[Spring实战](https://book.douban.com/subject/26767354/)


