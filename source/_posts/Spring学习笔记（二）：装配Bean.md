---
title: Spring学习笔记（二）：装配Bean
date: 2017-07-17 11:32:26
categories: Spring学习笔记
tags: Spring装配Bean
description: 上篇简单介绍了Spring的一些常用基础功能，通过XML把一个Bean注入到另一个Bean中，但是Spring还有其他更好的方式完成这项枯燥的工作，本篇将介绍Spring装配Bean的几种方法。
---
# 前言 #
---
在Spring中，对象无需自己查找或创建与其所关联的其他对象，相反，容器负责把需要相互协作的对象引用赋予各个对象。创建应用对象之间的协作关系的行为通常称为装配，这也是依赖注入的本质，本篇将介绍使用Spring装配bean的基础知识。

# 正文 #
---
在Spring中装配bean有多种方式：
## Spring配置的可选方案 ##
在上篇描述中，Spring容器负责创建应用程序中的bean并通过DI来协调这些对象之间的关系，作为开发人员，你需要告诉Spring要创建哪些bean并且如何将它们装配在一起。  
Spring提供了三种主要的装配机制：

> - 在XML中进行显示配置。
> - 在java中进行显示配置。
> - 隐式的bean发现机制和自动装配。

这三种配置可以随意搭配，但是我们在工作中建议用的是第三种自动装配机智。显示配置越少越好。而也有些时候我们不得不使用显示配置，不如有些代码不是你来维护的，但你需要为这些代码配置bean的时候。  
首先我们先看一下Spring的自动化配置。

## 自动化装配bean ##
Spring从两个角度来实现自动化装配：

>- 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
>- 自动装配（autowiring）：Spring自动满足bean之间的依赖。

组件扫描和自动装配在一起能够使你的显示配置降低到最少。

### 创建可被发现的bean ###
为了阐述组件扫描和装配，我们需要创建几个bean，比如在一个印象系统中的组件。首先创建CompactDisc类，Spring发现它并将其创建一个bean，然后创建一个CDPlayer类，让Spring发现它，并将CompactDiscbean注入进来。
首先我们定义一个CD的接口：
	
	package soundsystem;
	public interface CompactDisc{
		void paly();
	}

我们还需要一个CompactDisc的实现，并且带有`@Component`注解：
	
	package soundsystem
	
	@Component
	public class SgtPeppers implements CompactDisc{
	
		public void play(){
			System.out.println("play music");
		}
	}

上面SgtPeppers类中使用了`@Component`注解，这个注解表明该类会作为组件类，并告诉Spring为这个类创建bean，而没有必要显示配置bean。  
但是Spring的组件扫描默认是不启用的，我们还需要显示配置一下Spring，从而命令Spring去寻找带有 `@Component` 注解的类，并为其创建bean，下面代码展示了完成这项任务最简洁的配置：

	package soundsystem;
	@Configuration
	@ComponentScan
	public class CDPlayerConfig{
	}

CDPlayerConfig类通过java代码定义了Spring的装配规则，与上一篇中通过java配置的代码不同的是我们并没有声明任何bean，只不过它使用了@ComponentScan注解，这个注解能够在Spring中启用组件扫描。  
如果没有其他配置的话，@ComponentScan默认会扫描与配置类相同的包，CDPlayerConfig类位于soundsystem包中，因此Spring将会扫描这个包以及这个包下的所有子包，查找带有@Component注解的类，这样就能发现CompactDisc类，并会在Spring中自动为其创建一个bean。  
当然你也可以使用XML来启用组件扫描：

	<context:component-scan base-package="soundsystem"/>

这样，Spring就创建了SgtPeppersbean，如果想要把这个bean注入到目标bean中只需要使用@Autowired就可以了：

	@AutoWired
	private CompactDisc cd;

### 为组件扫描的bean命名 ###

现在，我们来探讨一下@ComponentScan和@Component。
Spring应用上下文中所有的bean都会给定一个ID，在上面的例子中，虽然我们没有明确为SgtPeppersbean设置iD，但Spring会根据类名为其指定一个ID，就是将类名的第一个字母变为小写，那么SgtPeppersbean的ID就是sgtPeppersbean。  
如果想为这个bean设置不同的ID，比如想将这个bean标识为longlyHeartsClub,那么需要将SgtPeppersbean类的@Component注解配置如下：

	@Component("longlyHeartsClub")
	public class SgtPeppers implements CompactDisc{
		···
	}

>tip：还有一种为bean命名的方式，不使用@Component注解，而是使用java依赖注入规范中所提供的@Named注解为bean设置ID，但是在实际工作中很少使用，这里不再叙述。

### 设置组件扫描的基础包 ###
上面代码中我们使用@ComponentScan注解可以命令Spring扫描配置类所在包以及子包来发现组件。但是如果你想扫描不同的包或者想扫描多个基础包，又该怎么办呢？  比如我们想把配置类单独放在一个包中，与其他的应用代码区分开，如果是这样的话，那默认的基础包就不能满足要求了。  
如果要满足上述需求，你所需要做的就是在@ComponentScan的value属性中指明包的名称：

	@Configuration
	@ComponentScan("soundsystem")
	public class CDPlayerConfig{}

如果想更清晰地表明你所设置的基础包，那么你可以通过basePackages属性进行配置：
	
	@Configuration
	@ComponentScan(basePackages="soundsystem")
	public class CDPlayerConfig{}

如果你的配置文件在不同的基础包中，只需要将basePackages属性设置为要扫描包的一个数组即可：

	@Configuration
	@ComponentScan(basePackages={"soundsystem","video",···})
	public class CDPlayerConfig{}

在上述代码中基础包是以String类型表示的，但这种方法是类型不安全的，如果你重构代码的话，那么所指定的基础包可能就会出现错误了。  
另一种方法就是将其指定为包中所含的类或者接口：

	@Configuration
	@ComponentScan(basePackageClasses={CDPlayer.calss,"DVDPlayer.calss",···})
	public class CDPlayerConfig{}

可以看到，basepackages属性变成了basePackageClasses.而且我们不再使用String类型的名称来指定包，而是设置了类，这些类所在的包将作为组件扫描的基础包。  

### 通过为bean添加注释实现自动装配 ###
现在我们已经知道如何将一个类命名为一个bean，接下来就是如何自动装配他们了。  
自动装配就是就是让Spring自动满足bean依赖的一种方法，在满足依赖的过程中，会在Spring应用上下文中寻找匹配某个bean需求的其他bean。为了声明要进行自动装配。我们可以借助Spring的@Autowired注解。  

	package soundsystem;

	@Component
	public class CDPlayer implementsMediaPlayer{
		private CompactDisc cd;

		@Autowired
		public CDPlayer(CompactDisc cd){
			this.cd = cd;
		}

		public void play(){
			cd.play();
		}
	}	

在上述代码中CDPlayer类的构造器上添加了@Autowired注解，这表明当Spring创建CDPlayer bean的时候，会通过这个构造器來实例化并传入一个可设置的CompactDisc类型的bean。  
@Autowired不进能用在构造器上，还能用在属性的Setter方法上，不管是构造器还是Setter方法还是其他的方法，Spring都会尝试满足方法参数上所声明的依赖，加入有且只有一个bean匹配以来需求的话，那么这个bean将被装配进来。  
如果没有匹配的bean，那么应用上下文创建的时候，Spring就会抛出一个异常。为了避免异常的出现，你可以将@Autowired的required属性设置为false:
	
	@Autowired(required=false)
	public CDPlayer(CompactDisc cd){
		this.cd = cd;
	}

将required设置为false时，Spring会尝试执行自动装配，如果没有匹配的bean的话，Spring会让这个bean处于未装配的状态，但是把required属性设置为false时，如果在你的代码中没有进行null检查的话，这个处于未装配状态的属性有可能会出现NullPointerException。  
如果有多个bean能满足依赖关系的话，Spring将抛出一个异常，表明没有明确指定选择哪个bean进行自动装配。  

>tip：和@Component一样，@Autowired也有一个备胎：@Inject。@Inject注解来源于java依赖注入规范，在自动装配中，Spring同时支持@Autowired和@Inject，不过在工作中@Autowired更常见。  

## 通过java代码装配bean ##
尽管通过组件扫描和自动装配实现Spring的自动化配置是更为推荐的方式，但是有时候自动化配置的方案是行不通的，比如你想要将第三方库中的组件装配到你都应用中，在这种情况下是没有办法在它的类上添加@Component和@Autowired注解的，因此就不能是用自动化装配的方案了。  
在这种情况下，你必须要才用显示装配的方式。在进行显示装配的时候有两种可选的方案：java和XML。  
在进行显示配置时，javaConfig是更好的方案，因为它更强大、类型安全并且对重构友好。因为它就是java代码，就想应用程序中的其他java代码一样。  
同时，javaConfig与其他的java代码又有所不同，它与应用程序中的业务逻辑和领域代码是不同的。尽管它与其他的组件一样是用相同的语言，但javaConfig是配置代码，它不应该包含任何业务逻辑，javaConfig也不应该侵入到业务逻辑代码中。通常会将javaConfig放到单独的包中，使它与其他的应用程序逻辑分离开来。

### 创建配置类 ###
接下来，让我们通过javaConfig显示配置Spring。  
在上面的CDPlayerConfig.java中我们使用的就是javaConfig：
	
	package soundsystem;
	@Configuration
	@ComponentScan
	public class CDPlayerConfig{
	}

创建javaConfig类的关键在于为其添加@Configuration注解，@Configuration的注解表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。  
到此为止，我们都是依赖组件扫描來发现Spring应该创建的bean。而本节关注的重点是显示配置，我们将@ComponentScan注解去掉。  

移除了@ComponentScan注解，此时的CDPlayerConfig类就没有任何作用了。如果你强行运行程序的话就会出现BeanCreation-Exception异常。这是因为bean根本就没有创建。

### 声明简单的bean ###
要在javaConfig中声明bean，我们需要编写要给方法，这个方法创建所需要类型的实例，然后给这个方法添加@Bean注解：
	
	@Bean
	public CompactDisc sgtPeppers(){
		return new SgtPeppers();
	}

@Bean注解会告诉Spring这个方法将返回一个对象，该对象要注册为Spring应用上下文中的bean。方法体中包含了最终产生bean实例的逻辑。  
默认情况下，bean的ID与带有@Bean注解的方法名是一样的，上面代码中的bean的ID是sgtPeppers，如果想为其设置不同的名字，可以重命名该方法，通过name属性指定一个不同的名字：

	@Bean(name="lonelyHeartsClubBand")
	public CompactDisc sgtPeppers(){
		return new SgtPeppers();
	}

现在，我们借助@Bean命名了一个bean，那么我们怎么将这个bean注入到其他bean中呢？  

### 借助javaConfig实现注入 ###
前面我们声明了CompactDisc bean，它是一个非常简单的bean，并没有任何功能，我们需要声明一个CDPlayer bean，它依赖于CompactDisc。  
在javaConfig中装配bean的最简单的方式就是引用创建bean的方法。例如下面是一种声明CDPlayer的方案：
	
	@Bean
	public CDPlayer cdPlayer(){
		return new CDPlayer(sgtPeppers());
	}

cdPlayer()方法像sgtPeppers()方法一样，同样使用了@Bean注解，这表明这个方法会创建一个bean实例并将其注册到Spring应用上下文中。在这里cdPlayer()方法调用了需要传入CompactDisc对象的构造器來创建CDPlayer实例。  
看起来，CompactDisc是通过调用sgtPeppers()得到的，但实际并非如此，因为sgtPeppers()方法上添加了@Bean注解，Spring将会链接所有对它的调用，并确保直接返回该方法所创建的bean，而不是每次都对其进行实际的调用。比如假设你引入了一个其他的CDPlayer bean，它和之前的那个bean完全一样：

	@Bean
	public CDPlayer cdPlayer(){
		return new CDPlayer(sgtPeppers());
	}

	@Bean
	public CDPlayer anothierCDPlayer(){
		return new CDPlayer(sgtPeppers());
	}

假如对sgtPeppers()的调用就像其他的Java方法调用一样的话，那么每个CDPlayer实例都会有一个自己特有的SgtPeppers实例。  
但是，在软件领域中，我们完全可以将同一个SgtPeppers实例注入到任意数量的其他bean之中。默认情况下，Spring中的bean都是单例的，我们并没有必要为第二个CDPlayer bean创建完全相同的SgtPeppers实例。所以，Spring会拦截对sgtPeppers()的调用并确保返回的是Spring所创建的bean，也就是Spring本身在调用sgtPeppers()时所创建的CompactDiscbean。因此，两个CDPlayer bean会得到相同的SgtPeppers实例。  

## 通过XML装配bean ##
到此为止，我们已经让Spring自动发现和装配bean,而且还知道如何通过javaConfig显示装配bean。但是在装配bean的时候，还有一种可选的方案：XML配置。  
需要说明的是在实际工作中我们已经不再使用XML配置了，更多的是通过注解或者javaConfig的方式。

### 创建XML配置规范 ###

在是用javaConfig的时候，我们需要写@Configuration注解來告诉Spring这是一个javaConfig的配置，而在XML中我们要创建一个XML文件，并要以<beans>元素为根。  
最简单的Spring XML配置如下：
	
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context">
	<!-- 具体配置内容 -->
	</beans>

在使用XML时，需要在配置文件顶部声明多个XML文件，这些文件定义了配置Spring的XML元素。就这样我们就有了一个合法的SpringXML配置，不过，它是一个没有任何用处的配置，因为它还没有声明任何bean。接下来我们用XML配置來创建一个bean。

### 声明一个简单的bean ###

在XML中我们使用Spring-bean模式中的一个元素：<bean>。<bean>元素类似于javaConfig中的@Bean注解。我们可以按照如下的方式声明CompactDisc bean：

	<bean class="soundsystem.SgtPeppers" />

这里我们声明了一个简单的bean，创建这个bean的类通过class属性來指定。因为没有给定ID，所以这个bean将会根据全限定类名來进行命名。在本例中，bean的ID将会是“soundsystem.SgtPeppers#0”。其中，“#0”是一个计数的形式，用来区分相同类型的其他bean。如果你声明了另一个SgtPeppers,并且没有明确给定ID，那么它自动得到的ID将会是“soundsystem.SgtPeppers#1”。  
为了方便，更好的办法就是为每个bean设置一个名字：

	<bean id="compactDisc" class="soundsystem.SgtPeppers" />

现在，我们就声明了一个简单的bean。当Spring发现这个<bean>元素时，它将会调用SgtPeppers的默认构造器來创建bean。

### 借助构造器注入初始化bean ###
在Spring XML配置中，只有一种声明bean的方式：使用<bean>元素并指定class属性。Spring会从这里获取必要的信息来创建bean。  
但是，在XML中声明DI时，会有多种可选的配置方案和风格。具体到构造器注入，有两种基本的配置方案可供选择：

> - <constructor-arg>元素
> - 使用Spring3.0所引入的c-命名空间

两者的区别在很大程度就是是否冗长繁琐。
#### 构造器注入bean引用 ####
按照前面的定义，CDPlayer bean接收CompactDisc类型的构造器。现在我们已经声明了SgtPeppers bean，并且SgtPeppers类实现了CompactDisc接口，我们现在要做的就是在XML中声明CDPlayer并通过ID引用SgtPeppers：

	<bean id="cdPlayer" class="soundsystem.CDPlayer">
		<constructor-arg ref="compactDisc" />
	</bean>

当Spring遇到这个<bean>元素时，它会创建一个CDPlayer实例。<constructor-arg>元素会告知Spring要将一个ID为compactDisc的bean引用传递到CDPlayer的构造器中。  

你也可以使用Spring的c-命名空间。c-命名空间是在Spring 3.0中引入的，它是在XML中更为简洁地描述构造器参数的方式。要使用它的话，必须在XML的顶部声明其模式：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:p="http://www.springframework.org/schema/c"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd">
	···
	</beans>

在c-命名空间和模式声明之后，我们就可以使用它来声明构造器参数了：

	<bean id="cdPlayer" class="soundsystem.CDPlayer" c:cd-ref="compactDisc" />

这里我们使用了c-命名空间来声明构造器参数。

{% asset_img pic_01.png pic_01.png%}

属性名以“c:”开头，也就是命名空间的前缀。接下来就是要装配的构造器参数名，在此之后是“-ref”，这是一个命名的约定，它会告诉Spring，正在装配的是一个bean的引用，这个bean的名字是compactDisc，而不是字面量“compactDisc”。  
在上面的代码中我们使用了构造器参数的名称，还有一个可替代的方案：
	
	<bean id="cdPlayer" class="soundsystem.CDPlayer" c:_0-ref="compactDisc" />

这里的配置是吧参数的名字替换成了“0”，也就是参数的索引，因为XML中不允许数字作为属性的第一个字符，因此必须要添加一个下划线作为前缀。  
当构造器有多个参数的时候这种方式更为简便，如果只有一个构造器参数，我们还有另外一种方案：

	<bean id="cdPlayer" class="soundsystem.CDPlayer" c:_-ref="compactDisc" />

这里没有参数索引或则参数名，只有一个下划线，然后就是用"ref-"来表明正在装配的是一个引用。

#### 将字面量注入到构造器中 ####

现在我们所做的DI通常是指类型的装配，也就是将对象的引用装配到依赖于它们的其他对象中，而有时候，我们需要做的只是用一个字面量值来配置对象。如下：
	
	package soundsystem

	public class BlankDisc implements CompactDisc{
		
		private String title;
		private String artist;

		public BlankDisc(String title,String artist){
			this.title = title;
			this.artist = artist;
		}

		public void play(){
			System.out.println("Playing"+title+"by"+artist);
		}
	}

在SgtPeppers中，唱片名称和艺术家的名字都是硬编码的，但这个CompactDisc实现与之不同，它可以设置成任意的艺术家和唱片名，现在我们可以使用这个方案：

	<bean id="compactDisc" class="soundsystem.BlankDisc">
		<constructor-arg value="titleName" />
		<constructor-arg value="artistNmae" />
	</bean>

我们再次使用<constructor-arg>元素进行构造器参数的注入。但是这一次我们没有使用“ref”属性来引用其他的bean，而是使用了value属性，通过该属性表明给定的值要以字面量的形式注入到构造器之中。  

如果使用c-命名空间的花，这个例子变成了如下：

	<bean id="compactDisc" class="soundsystem.BlankDisc"
		c:_title="titleName"
		c:_artist="artistName" />

可以看到，装配字面量与装配引用的区别在于属性名中去掉了“-ref”后缀。与之类似，我们也可以通过参数索引装配相同的字面量值，如下所示：

	<bean id="compactDisc" class="soundsystem.BlankDisc"
		c:_0="titleName"
		c:_1="artistName" />

XML不允许某个元素的多个属性具有相同的名字。因此，如果有两个或更多的构造器参数的话，我们不能简单地使用下画线进行标示。但是如果只有一个构造器参数的话，我们就可以这样做了：
	
	<bean id="compactDisc" class="soundsystem.BlankDisc"
		c:_="titleName" />

#### 装配集合 ####
到现在为止，我们假设CompactDisc在定义时只包含了唱片名称和艺术家的名字。而现实中的CD还应该有不少于一首的歌曲，那么BlankDIsc就变成了下面的情况：
	
	import soundsystem.CompactDisc;

	public class BlankDisc implements CompactDisc {

		private String title;
		private String artist;
		private List<String> tracks;

		public BlankDisc(String title, String artist, List<String> tracks) {
    		this.title = title;
		    this.artist = artist;
		    this.tracks = tracks;
		}

		public void play() {
		    System.out.println("Playing " + title + " by " + artist);
		    for (String track : tracks) {
				System.out.println("-Track: " + track);
		    }
		}
	}

这个变更会对Spring如何配置bean产生影响，在声明bean的时候，我们必须要提供一个歌曲列表。

	<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="titleName" />
    <constructor-arg value="artistName" />
    <constructor-arg>
      <list>
        <value>songName_1</value>
        <value>songName_2</value>
        <value>songName_3</value>
		···
      </list>
    </constructor-arg>
	</bean>

其中，<list>元素是<constructor-arg>的子元素，这表明一个包含值的列表将会传递到构造器中。其中，<value>元素用来指定列表中的每个元素。与之类似，我们也可以使用<ref>元素替代<value>。

	<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="titleName" />
    <constructor-arg value="artistName" />
    <constructor-arg>
      <list>
        <ref bean="songName_1" />
        <ref bean="songName_2" />
        <ref bean="songName_3" />
		···
      </list>
    </constructor-arg>
	</bean>

我们也可以按照同样的方式使用<set>元素：

	<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="titleName" />
    <constructor-arg value="artistName" />
    <constructor-arg>
      <set>
        <value>songName_1</value>
        <value>songName_2</value>
        <value>songName_3</value>
		···
      </set>
    </constructor-arg>
	</bean>

<set>和<list>元素的区别不大，其中最重要的不同在于当Spring创建要装配的集合时，所创建的是java.util.Set还是java.util.List。如果是Set的话，所有重复的值都会被忽略掉，存放顺序也不会得以保证。不过无论在哪种情况下，<set>或<list>都可以用来装配List、Set甚至数组。  
在装配集合方面，<constructor-arg>比c-命名空间的属性更有优势。目前，使用c-命名空间的属性无法实现装配集合的功能。

##  导入和混合配置 ##

在Spring应用中，我们可能同时使用自动化和显示配置。在Spring中这些配置方案都不是互斥的，你可以将javaConfig的组件扫描和自动化装配或XML配置混合在一起。  

### 在javaConfig中引用XML配置 ###

假设现在有两个javaConfig配置类，我们可以在一个javaConfig中使用@Import注解导入另一个配置类：

	@Configuration
	@Import(CDConfig.class)	//导入CDConfiug.java
	public class CDPlayerConfig{···}


或者采用一个更好的方法，创建一个更高级的配置类，在这个类中使用@Import将两个配置类组合在一起：

	@Configuration
	@Import(CDConfig.class，CDPlayerConfig.class)	//导入CDConfiug.java、CDPlayerConfig.java
	public class SoundSystemConfig{···}	

现在，如果我们希望在javaConfig类导入XML配置呢，我们可以用@ImportResource注解来实现：

	@Configuration
	@Import(CDConfig.class)	//导入CDConfiug.java
	@ImportResource("classPath:cd-config.xml")	//导入cd-config.xml
	public class SoundSystemConfig{···}

### 在XML配置中引入javaConfig ###
上面说了如何通过@Import和@ImportResource来拆分JavaConfig类。那么我们是否也可以把XML拆分呢。  
事实上，答案并非如此，<import>元素只能导入其他的XML配置文件，并没有XML元素能导入javaConfig类。  
但是，有一个元素能够用来将java配置导入到XML配置中：<bean>元素。为了将javaConfig类导入到XML配置中，我们可以这样声明bean：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:c="http://www.springframework.org/schema/c"
		xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd">

		<bean class="soundsystem.CDConfig" />
		<bean id="cdPlayer" class="soundsystem.CDPlayer" c:cd-ref="compactDisc" />
	</beans>

采用这样的方式，两种配置————其中一个使用XML描述，另一个使用java描述————被组合在了一起。类似地，你可能还希望创建一个更高层次的配置文件，这个文件不声明任何的bean，只是负责将两个或更多的配置组合起来。例如，你可以将CDConfig bean从之前的XML文件中移除掉，而是使用第三个配置文件将这两个组合在一起：
	
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:c="http://www.springframework.org/schema/c"
		xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd">
	
		<bean class="soundsystem.CDConfig" />
		<import resource="cdplayer-config.xml" />
	</beans>

不管使用JavaConfig还是使用XML进行装配，我通常都会创建一个根配置（root configuration），也就是这里展现的这样，这个配置会将两个或更多的装配类和/或XML文件组合起来。我也会在根配置中启用组件扫描（通过<context:component-scan>或@ComponentScan）

# 总结 #
---
我们看到了在Spring中装配bean的三种主要方式：自动化配置、基于Java的显式配置以及基于XML的显式配置。建议尽可能使用自动化配置，以避免显式配置所带来的维护成本。但是，如果你确实需要显式配置Spring的话，应该优先选择基于Java的配置，它比基于XML的配置更加强大、类型安全并且易于重构。

# 参考 #
---

[Spring教程](http://www.yiibai.com/spring/)