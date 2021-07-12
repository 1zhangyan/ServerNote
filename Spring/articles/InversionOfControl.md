# 控制反转
> [InversionOfControl原文链接](https://martinfowler.com/bliki/InversionOfControl.html)  
当你扩展框架的时候，控制反转是你经常遇到的现象。它也经常被视为是框架的典型特征。  
来看一个简单的例子。我正在写个命令行查询程序--获取用户信息，我有时可能这样干：    
```ruby
puts 'What is your name?'
name = gets
process_name(name)
puts 'What is your quest?'
quest = gets
process_quest(quest)
```

在这段交互中，我的程序是控制方：我的程序决定什么时候询问问题，什么时候读取回复以及什么时候处理这些结果。  

但是当我准备去使用一个窗口系统实现同样的功能，我需要先配置一个窗口。
```ruby
require 'tk'
root = TkRoot.new()
name_label = TkLabel.new() {text "What is Your Name?"}
name_label.pack
name = TkEntry.new(root).pack
name.bind("FocusOut") {process_name(name)}
quest_label = TkLabel.new() {text "What is Your Quest?"}
quest_label.pack
quest = TkEntry.new(root).pack
quest.bind("FocusOut") {process_quest(quest)}
Tk.mainloop()
```
这两个程序在控制流上有巨大的不同之处，特别是在控制何时调用process_name和process_quest这两个函数。在命令行方式中我控制着这两个函数的调用。而在窗口方式中，通过创建时绑定，我把控制权交给了窗口系统，由它决定何时调用我的函数。控制发生了反转，是这个框架调用了我，而不是我调用了框架。这种现象称为控制反转（也叫好莱坞原则 "Don't call us, we'll call you"）。  

> 框架的一个重要特征就是用户定义的用来填充框架的函数常常是被框架本身调用的，而非用户程序调用。框架扮演的角色通常是使应用行为协调有序的主程序。反转控制让框架有了扩展的力量。用户定义的方法用来填充框架为一个应用定义好的规则。  --Ralph Johnson and Brian Foote   

控制反转是框架区别与库的一个重要部分。库在本质上是一组可供你调用的函数，如今常被组织成类。每次调用会做一些工作，并且最终会将控制权返回给客户。  

框架是一些抽象设计的具体表现，并且内建了更多的行为。使用框架时，你需要做的是将行为插入框架的不同地方而不是继承或者插入自己的类。框架代码可以在插入点调用的你代码。  

有许多种方式插入要调用的代码。在上面的例子中，我门在输入的地方调用了一个绑定在输入入口的方法，传递事件名称和一个lambda表达式作为参数。一旦检测到输入事件，它就会调用闭包中的方法，使用闭包中的方法是很方便的事情，但是很多语言不支持。  

另一种方式是让框架定义事件，让客户端代码订阅这个事件。.net平台是个非常好的例子，它在语言上支持用户在小部件上绑定事件，之后可以使用委托在事件上绑定方法。  

上面两种方法在单例中作用很好，但是有时候你需要在一个单元内结合几个需要的方法调用作为拓展。在这种情况下，框架可以定义接口，客户代码必须实现这些接口完成相关调用。