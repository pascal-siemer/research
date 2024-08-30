# Research on Derivable Interfaces

<sub>the term 'derivable interface' is inspired by Rusts derivable Traits and Rusts and Haskells derive/deriving keywords</sub>  

If we look into languages like Haskell and Rust, one feature might be confusing at first:  
how can they implement features like standard string formatting ([Haskell] Show, [Rust] Display) on custom data types?  

At first it seems like magic but if you get to the core of it, its actually default implementations on interfaces-likes being put on our custom data.  
In Rusts case its more of the compiler auto-generating default implementations for these interfaces-likes (traits), but that is more a detail than a difference.  

So: why does this post exist?  
I found a way to implement such a mechanism in csharp using its default featureset.  

## Default Implementations on Interfaces
Modern csharp has the ability to implement default implementations for interface members which do not need to be implemented in its implementing classes.  
Wow! That sound just like what we want!  
Well.. not really.  
The default implementation is only available when accessing an object over its interface. The implementation itself does not exist on the class.  
My guess is that has been done to deal with the diamond-problem.  

But luckily, csharp has a mechanism to implement functions on classes from outside of the class.  
What we need is an **Extension** on said 'derivable interface' that makes these default implementations accessible by default.  
So now we have to deal with the diamond problem? And how do these extensions deal with actual implementations on these classes?  
In Short: When overwriting the default implementation, this implementation will always be used, even with an extension. More on that later.  
Also: this solution deals with the diamond problem in preventing you from using the extension method on a class if two equal extension methods have been found.  
This is basically what Rust seems to be doing in this case.  

Here is an code example of a "derivable interface":  

    public interface Displayable {
        public string Display() {
            ... // default implementation
        }
    }

    public static class DisplayableExtensions {
        
        public static string Display(this Displayable self) {
            return self.Display;
        }

        
        // or with generics: 
        public static string Display<Self>(this Self self) where Self: Displayable {
            return self.Display;
        }           
    }

    public class Thing : Displayable {
        ...
    }


## test cases:  
#### [OK] implementation on interface, not on class.
uses default interface implementation callable by class directly.

#### [OK] implementation on interface, also on class.
uses class method.

#### [OK] diamond-problem: two extensions registering the same Function onto the same class.
interface implementations has to be called explicitly by static Extension method (`extension.method(this)`).

#### [OK] implementation on interface, also on class that inherits an equal function.
inherit class methods are always preferred!

#### [OK] implementing "derivable interface" twice (different generics?). --> Appending the function to the class is more difficult with the full generic approach.
works though appending the extension to a class can be done wrong.
when implementing a gerneric interface twice, the generics have to be specified when calling the method.


## results:

class methods always have a higher priority in method resolution. when having multiple extensions with the same name, they have to be called over the extension class explicitly.
method resolution order seems to be: class > parent classes > extension


## Ideas:  

- maybe this pattern can be simplified by using attributes and code generation? how would IDEs work with generated code?
