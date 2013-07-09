# The Elements of Programmatic Style

## Introduction

The purpose of this guide is to ensure existing and future iOS and Mac products conform to a consistent and well-designed convention. This will make applications easier to build and debug for both newcomers and existing members alike. In this document, we will be using Shopify (aka Jaded Pixel) as an organization name, but the same rules can apply to any organization, even to single developers.

Many of the conventions in this guide are common to existing Cocoa style guides (see the References section below), and try to combine the best of those worlds.

This document aims for excruciating clarity, attempting to document all possible aspects of iOS application development. If an aspect of coding style is not addressed here, please inform the authors and have the document updated accordingly.

This document also strives to include the best and most modern features of the Objective-C programming language, Cocoa frameworks, and the iOS SDK in general, so that excessive baggage does not creep in. As the environment evolves, so too must this guide.

### Terminology

First, some quick terminology.

* `(` and `)` are *parentheses*
* `{` and `}` are *braces*.
* `[` and `]` are *brackets*.
* `<` and `>` are *angle brackets*.
* `*` is an *asterisk* or *star* (or *pointer thingy* `:)`).
* `_` is an *underscore*.

### About Performance

Some of these guidelines may seem counter-perfomance (i.e., these conventions may appear to hurt performance), but in fact, they are in place to *help* performance: developer performance. Most runtime performance hits as a result to this guide will be negligible, but the gains to the developers will be substantial.

If any of the guidelines causes a substantial performance hit, it can be dealt with on a case basis, and only after it has been thoroughly profiled.

## Naming

### In General

When giving a name to any kind of symbol, you should strive to make it descriptive over terse. We have text editors capable of auto-completing symbol names, and we expect to make great usage of this. This extends even to iterator variables: those should be named accordingly (even if it is just `index`, but especially in the case of nested loops, `i`, `j`, and `k` will have your head spinning in no time).

    NSInteger currentColumn;

The Cocoa naming conventions call for camel-casing all symbol names (with preprocessor bits being the only exclusion):

    #define X_OFFSET 25.0f
    NSInteger index = [SomeClassName classMethod];

Do not abbreviate symbols and do not use acronyms unless they are extremely common (in which case you use uppercase for each letter of the acronym) like `URL` or `PNG`. See the [Coding Guidelines for Cocoa - Acceptable Abbreviations and Acronyms][apple2] for a list of acceptable abbreviations and acronyms.

### Files

Files should be given descriptive, camel-cased names, with the initial letter being uppercased. When creating classes in Xcode, this is the default behavior, as the file names match the class names they contain.

Files should be prefixed with the project prefix which should be the initials of the organization (in our case, this is `JPX` for Jaded Pixel). The prefix should contain at least three letters, because two-letter prefixes are reserved for Apple framework classes. See [Programming with Objective-C - Class Names Must Be Unique Across an Entire App][apple3] for more information. 

    JPXClassName.h
    JPXClassName.m

**Note**: A file name should **never** be named with the same prefix as an Apple provided class, with the exception of *Categories*, whose file naming conventions are described below.

### Classes

Class names must be indicative of their class hierarchy. That is, from the class name, you should be able to correctly guess exactly what kind of class it is. Do not rely on the folder/group structure in Xcode to indicate this, as the text editor itself ignores in which folder the class appears.

    JPXProductsTableViewController

The name above is descriptive and immediately indicates to the developer that this class displays products and is a subclass of `UITableViewController`.

Controller classes should contain `Controller` at the end of their name, view classes should contain `View` at the end of their name (e.g. `JPXProductsTableViewCell`), and model classes should generally contain neither, but still indicate what they are (avoid the use of `Model` in the name as it is redundant).

**Note**: A class should **never** be named with the same prefix as an Apple provided class. This is implicitly guaranteed if you use a three-letter prefix (as suggested above).

### Protocols

Protocols should be named like the classes they pertain to, additionally appending the protocol role. If there is no distinct role, appending `Protocol` is sufficient. The goal is to be able to tell at a glance what the symbol represents.

    JPXCollectionViewDelegate
    JPXCollectionViewDataSource
    JPXReaperProtocol

Methods in a protocol are implicitly `@required`. It is often desirable to have `@optional` methods as well (e.g., when there is a default value for data source method). `@required` methods should be grouped together at the top, `@optional` methods should be grouped together at the bottom.

### Categories

Category file names should indicate the name of the class being extended, followed by a `+`, followed by the name of the category. This is the default behaviour as of Xcode 4.6.3.

When naming the category, avoid using generic names like "Additions", instead be more descriptive of what the category adds to the class. If the category methods are a grab bag, then use `Utilities`. The category name must also be prefixed correctly.

    NSString+JPXUtilities.h
    NSString+JPXUtilities.m

Categories are used to extend existing classes at runtime without subclassing, and we can make liberal use of these. But categories **must not** be used to override existing methods. Therefore, **all** methods of the category must be appropriately prefixed. Use the project prefix followed by `_` as the method prefix.

    - (void)JPX_doSomething;

### Variables

Variables are to be named in camel-case and begin with a lowercase letter. This applies to instance, static, and local variables. Block variables are treated the same as local scope variables in terms of naming conventions.

When declaring object or any other pointer type variables, the star `*` belongs to the variable name, not the class name or type.

Keep each variable declaration on its own line, even if subsequent declarations are of the same type.

    NSString *username;
    NSString *password;

#### Instance Variables

Instance variables should be named like normal variables, except with a leading underscore `_`.

Do not declare them in the public `@interface` of a class, as instance variables are an implementation detail. Instead, declare them in the `@implementation` block or declare private properties in the implementation and synthesize the instance variables. See the section below for more information on interfaces and implementation.

#### Constants

Private constants should be declared with the `static` and `const` keywords and should start with the project prefix. They should **only** be declared in implementation files. `static` ensures that the symbol is only visible inside the implemention file. `const` ensures that no code in the implementation file changes the value of the symbol.

    static NSString *const JPXAnimationDurationKey = @"JPXAnimationDurationKey";
    
Public constants should be declared in the header and should be initialized in the implementation file. The actual value of a public constant is an implementation detail. Because of this split, the declaration must use `extern` to indicate that the symbol in the header is initialized somewhere else. The name of a public constant begins with the name of the class in which the constant is declared (in the example below `JPXProduct`), followed by a descriptive name.

    extern NSString *const JPXProductDataKey;                   // In file JPXProduct.h
    NSString *const JPXProductDataKey = @"JPXProductDataKey";   // In file JPXProduct.m

Both private and public constants should be initialized with their symbol name. This effectively prevents conflicts with other constants.

#### Enumerations

Enumerations should be used whenever a variety of integer values could be used (usually as some kind of discriminate grouping). Enumerations should be typedef'd with the `NS_ENUM()` preprocessor macro and given a name in a similar style to their related class. The enum values should be named similarly, including the name of the type, with the actual type appended.

    typedef NS_ENUM(NSInteger, JPXProductPlacementType) {
        JPXProductPlacementTypeTop,
        JPXProductPlacementTypeCenter,
        JPXProductPlacementTypeBottom
    };

Even though enumeration types are just integers, they should always be treated as their own type (e.g. `JPXProductPlacementType`, and never just `int`). This gives the symbol extra context instead of just some random, unrelated type. The use of `NS_ENUM()` allows the compiler to provide code-completion for enumerations and to emit more warnings if the enumeration is used inappropriately. For more information about `NS_ENUM()`, see [NSHipster - NS_ENUM & NS_OPTIONS][nshipster1]. 

You should never use a plain integer when an enumeration value is expected.

    cell.placementType = 1;                                // Bad
    cell.placementType = JPXProductPlacementTypeCenter;    // Good

If the number the enumeration value resolves to ever changes, you get the new value for free!

#### #define

`#define` should only be used for small, internal uses where one of the above symbol types seems like overkill. `#define` is the poor-man's constant. Defines should be all uppercase, with underscores used as a delimiter. 

    #define CELL_X_OFFSET 25.0f

### Methods

Methods should be given descriptive names, where clarity is the most important feature. Names should be camel-cased with the initial letter lowercased.

Make method names as long as necessary, naming with hints towards the condition, cause, and return value of the method. Methods should read like sentences, and the developer should exactly grasp the purpose of the method and invocation simply by reading its name.

The single parts of the selector (i.e., the words between the colons) should be named without the use of `and`, `with`, etc. These conjunctions are not necessary and become excessive after only a few instances. Exceptions can be made for the first part of the selector.

    - (id)initWithFrame:(CGRect)frame;                                         // Good
    - (id)initWithFrame:(CGRect)frame backgroundColor:(UIColor *)color;        // Good
    - (id)initWithFrame:(CGRect)frame andBackgroundColor:(UIColor *)color;     // Bad

When creating new methods which provide more options to existing methods, list all the original parameters in your new method *first*, before adding additional parameters (as in `-initWithFrame:backgroundColor:` above). The only exception to this rule is when the final parameter of the existing method accepts a Block. In this case, the new method must keep the Block paramter as the final parameter.

    - (void)haveAPartyWithFriends:(NSArray *)friends cleanUpHandler:(JPXCleanUpHandler)cleanUpHandler;
    - (void)haveAPartyWithFriends:(NSArray *)friends 
                cateringAvailable:(BOOL)cateringAvailable 
                   cleanUpHandler:(JPXCleanUpHandler)cleanUpHandler;

Selector signatures (both interface and implementation) must follow the standard usage of whitespace as demonstrated throughout this guide.

1. The method scope (`-` and `+` for instance or class method, respectively), followed by a space.
2. The return type of the method, with a space between the type and the star (this is really just like a cast, and all casts must follow this guideline). If there is no star, then there is no space. Double-star casts (e.g. `(NSError **)`) should not have a second space.
3. The start of the selector should directly follow the cast *without whitespace*.
4. If the method takes parameters, they should follow a colon, their cast (with no space between the colon and the cast, and no space between the cast and the argument name).
5. After the last parameter in the declaration, there should be a semicolon in the declaration.
6. After the last parameter in the implementation, there should be a space and then an opening brace **on the same line**.

    // Declaration in header
    - (NSString *)stringByMashingFirstName:(NSString *)firstName lastName:(NSString *)lastName;
    
    // Implementation in implementation file
    - (NSString *)stringByMashingFirstName:(NSString *)firstName lastName:(NSString *)lastName {
        // Do something ...
    }

Each method declaration should appear on its own line, below declared `@property` entries. Group related methods together and add a single line of whitespace between groups.

Finally, do not be abusive with the compiler:

* A return type must be provided, even though this is not required by the compiler.
* All parameters must be named, even though this is not required by the compiler.
* All parameters must be typed, even though this is not required by the compiler.

    - validButStupidlyNamedSelector:::;    // Bad

####Functions

Always name all arguments even in the header declaration. This assists with code completion and makes it easier for the developer to know which parameters do what.

	void someFunction(NSString *); // compiles, but so dirty
	void someBetterFunction(NSString *name); // delightful and satisfying.

Whitespace
----------

Whitespace is free. We all have large, high resolution displays. It's good to let the code breathe so it's more readable. Try to keep logical bits and pieces together, and use vertical whitespace to achieve rhythm and pacing.

###General

Use tabs instead of spaces (the default setting, which uses a tab size of 4). Xcode will try to assist you in keeping things properly indented. Follow its suggestions (pro tip: if Xcode is being funky about how it indents things, you've probably got a syntax error somewhere).

There is typically no reason to hard-wrap lines, even though Objective-C tends to be a verbose language, we also have wide diplays, so there is rarely a need to wrap the lines manually. Turning on word-wrap helps for smaller displays.

If you feel the need to wrap a method line, either the signature or an invocation, you may do so but make sure the parameter colons line up properly (Xcode will try its best to do this for you), but generally this isn't necessary.


###Control Structures

Control structures like `if`, `for`, `while`, etc all require one space between the keyword and the open paren, and one space between the closing paren and the opening brace. There should be no padding spaces inside the parens before the actual condition.

	if (someCondition == otherThing) {
		NSLog(@"Great success!", nil);
	}

Assignments inside conditionals should be avoided, but in the case of `-init`, should be wrapped in double parentheses, as recommended by the Clang Fixit system.

Using control structures without braces should be avoided, as this can lead to hard to detect errors if additional lines are later added. If you decide to go without braces, prefer to keep your code all on one line:

	if (!condition) return; // bailing early.

####Comparing against `nil` and buddies 

When comparing a variable for equality against `nil` or some constant, put the constant value first. This avoids accidentally forgetting the second `=` sign and therefore avoids an accidental assignment inside the conditional. Though the Clang compiler will warn against such occurrences, it's just better practice to avoid the mistakes altogether.

	if (hopefullyExistingObject = nil) { ... // oops! bad.
	if (nil == hopefullyExistingObject) { ... // much better. No chance for bugs here.


Project Conventions
-----------------

###Keep public API simple

The Public API is what's exposed in your header files. These headers should only include the very minimal set of methods which can be used by outside classes. Any internal methods which won't either be invoked by outside classes, or overridden by sublcasses, should not be put in the `.h` file. This includes `@property` declarations. Public properties belong in the headers, but private ones should be declared in the **class extension**.

This also means header files should not contain instance variable declarations, as they are really an implementation detail. There are two other ways to declare instance variables (without dropping down to the runtime level).

1. Variables can be synthesized with the `@synthesize` and a matching property name (and synthesized variables can even be given a custom name like so `@synthesize myPropertyName = _myPropertyName`. The propery is still `myPropertyName`, but the actual instance variable created has a leading underscore).

2. Declaring instance variables in the `@implementation`. This is just like declaring them in the `@interface`, except they're no longer visible to consuming classes or subclasses.

		@implementation MyClass {
			NSString *_internalVariable;
		}

Of the two methods, using properties (even if just declared in the Class Extension) is better better because it generates getters and setters, which give us a common place to add extra code. If bare ivars are used and a change must be made, then we've got extra work to be done.

###Keep implementations ordered

This is an easy one. Try to keep methods in the implementation files grouped together logically, instead of strewn about the file. If there are a bunch of delegate methods implemented, put those together, and use a `#pragma mark - SomeDelegateProtocol methods` to mark the beginning of a section of methods.

For subclasses, try to keep your overrides grouped as well, in order of the hierarchy. For example, keep `NSObject` override methods first, then say, `UIViewController` overrides, then the subclass's methods.

###Class extensions

Class extensions are how internal methods and properties should be hidden out of the public interface and header file. They are kind of like anonymous categories. They are to be placed in the `.m` file before the main `@implementation` block. For a class named `ShopTableViewController`, the class extension might look like this:

	@interface ShopTableViewController ()

	@property (nonatomic, strong) NSString *internalString;

	- (void)doSomethingInternal;

	@end

The methods and properties declared in this extension become synthesized and implemented just like any other method or property in a public interface. There is no need to create a second, named implementation block.

###Comments

Comments are good in the public interface to describe what the public APIs do and what the methods' parameters and return values need to be (e.g. "This parameter must not be nil!").

For general code comments, they are often not as necessary. Rather than lots of comments, the methods should be descriptively named, as mentioned above. Comments in implementations can quickly become out of date, and are often reduntant.

Comments which explain *why* a chunk of code exists are much better than comments which explain *what* a chunk of code does. Most programmers can follow along a block of code fairly easily without comments, but the real benefit is explaining something like "We're shifting the bits here to set a flag, which is needed so networking turns on", or something to that effect.




###Being a good Cocoa citizen

Being a good Cocoa citizen means following the conventions of the environment. We can get lots accomplished with the provided libraries. External frameworks like Three20 should be considered kind of harmful because they impose conventions from a different programming environment.

####Blocks

When implementing a callback mechanism, use Block objects instead of delegation wherever possible. These cut down on the verbiage of having to declare a delegate protocol, then marking your class as conforming to the protocol, then implementing the delegate methods.

With Block objects, they are typically `#typedef`'d in a class header, and then implemented inline as they are needed. Much simpler.

Use caution with Block objects, as they automatically retain objects referenced inside them. Usually this isn't too much of a problem as they are also released when the Block is destroyed, but it can lead to retain cycles. Consider using `__weak SomeClass weakObject = someStrongObject` and referencing the weak version instead in the Block object to avoid such a cycle (if `someStrongObject` retains the Block object).

####Delegates

Delegates are useful when fine granularity is needed in callbacks, or when multiple objects might be using the same callbacks, although they should generally be avoided in favour of Block object handlers.

####Use Cocoa "primitives"

Try to use the declared Cocoa "primitives" instead of their plain C counterparts. This means using `NSInteger` and `NSUInteger` instead of `int` and `unsigned int`, or using `CGFloat` instead of `float`.

We do this because it gives better flexibility if Apple ever changes what they decide an `NSInteger` should refer to. The Cocoa types are really just typedefs which evaluate to different primative types depending on the archictecture they are compiled for. If Apple ever move iOS to a 64 bit platform, all uses of `NSInteger` will automatically be updated on the next compile to use the proper types. We won't have to modify our code nearly as much for any possible future processor architecture changes.

It also means we should use types like `NSTimeInterval`, which really evaluates to a `double`, but using the Apple provided types are better because they give more context as to what the variable represents.


####Exceptions

Throwing exceptions in Cocoa is reserved for programmer error. If there is a possibility of something going wrong at runtime, your API should require an `NSError` pointer pointer and the method should return `NO` or `nil` on error condition, filling the error pointer too. Exceptions should not be used.

If your code crashes on an `Uncaught Exception`, this means there is something incorrect in your code (i.e. programmer error) and you need to fix this. A typical example would be an ArrayOutOfBoundsException. This is a mistake by the programmer, not a runtime error condition.

###ARC

All new projects should use Automatic Reference Counting. Existing projects should be upgraded as soon as possible.

For projects supporting iOS 5+, we should be using `weak` and `strong` properties. Though `retain` and `strong` are synonyms in ARC, `strong` should be used instead as it gives a better and more consistent indication of what the property is doing in terms of the object graph. It's also more consistent with the storage types (like `__weak`, `__strong`, etc.).


###Warnings and Errors

Warnings should be treated as errors, both by you and by the compiler. A warning is just a bug that maybe hasn't happened yet.

All Xcode projects should have the `-Werror` compiler flag turned on and left on. If we turn this flag on from the beginning, then fixing warnings should not be a problem. If we turn it on later in the project, we'll have more work to do.

Fix all the warnings as soon as they happen so they don't get out of control. 


###Xibs and Interface Builder

Interface Builder documents can be really useful and allow us to quickly prototype our interfaces, but they can usually only get us 90% of the way, with the other 10% being impossible without using code.

Xibs and Storyboard files are very difficult to merge if two or more developers have worked on them on different branches, causing massive headaches.

Due to these issues, using Interface Builder documents should be avoided for everything except quick prototyping (which will be later thrown out).

###Base SDK and Deployment Target

The project's `Base SDK` should always be set to `Latest iOS Version`, so we'll always be linking against the newest SDK code and symbols.

Our Deployment Target depends on the oldest possible OS version we'd like to support, e.g. only supporting back to iOS 5.0.


Being a good boyscout
---------------------

Finally, do your best to always be improving the code. Leave it in a better state than how you found it. If you notice a class is messy, then clean it up and make it adhere to this style guide.

References
---------

* [Google] [google]
* [Marcus Zarra] [zarra]
* [Apple] [apple]
* [Code Commandments] [commandments]

[google]: http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml
[zarra]: http://www.cimgf.com/zds-code-style-guide/
[commandments]: http://ironwolf.dangerousgames.com/blog/archives/913
[apple]: http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html
[apple2]: http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/APIAbbreviations.html#//apple_ref/doc/uid/20001285-BCIHCGAE
[apple3]: http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html
[nshipster1]: http://nshipster.com/ns_enum-ns_options/
