# Ruby lang

## Programmer-Centred Philosophy

Ruby is designed to make programmers happy.

==Matz designed Ruby with an emphasis on the needs of humans rather than computers==, which makes it intuitive and easy to read. It emphasises natural language elements, significant whitespace, and common idioms, so Ruby code often reads like English sentences.

Ruby is designed to be flexible to allow programmers to have multiple ways to do the same thing. This gives you the freedom to write code that best suits your thinking process.

## OO

Ruby is pure OO. ==Everything is an object, and all objects have methods==. Even basic data types like integers are objects with methods.

## Concepts

object 形體

class 形組 / 形类

type 形式

attribute 形態 / 體態

method 形變 / 體變

OO 以「形體」為主導的模式。

**Encapsulation** describes the fact that an object contains both its own data and the methods required to manipulate that data.

**Polymorphism** describes the ability to have ==different classes containing methods with the same name==. The same “message” can be sent to different objects, and ==each different object responds differently to the same message with its own special method==. For example, the same messsage or command "talk" is sent to a cat and a dog, and the cat and the dog responds differently to this message.

### Metaphors

符號「`.`」其實相當於英文裡的「'」或者中文裡的「的」。

一個 Function 就像一台小機器，被調用（call）時，就會進行「來料加工」，然後根據「指定規格」（-> type）返回（return）一個結果。

那麼，我想，一個 Struct 或一個 Class 就像一個個的工廠🏭。

Variable 或 Constant 就是些容器/載具，而且可能是透明材質的，用來裝東西（value）。

被大括號 `{}` 包起來的叫做 code block「碼塊」，裡面是實際上會執行的內容。





	alias_method :after_reset, :resets
	
	delegate :set, :reset, to: :instance



```ruby
def generated_attribute_methods
	@generated_attribute_methods ||= Module.new.tap { | mod | include mod }
end
```
