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

``` objc
NSInteger currentColumn;
```

The Cocoa naming conventions call for camel-casing all symbol names (with preprocessor bits being the only exclusion):

``` objc
#define X_OFFSET 25.0f
NSInteger index = [SomeClassName classMethod];
```

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

``` objc
- (void)JPX_doSomething;
```

### Variables

Variables are to be named in camel-case and begin with a lowercase letter. This applies to instance, static, and local variables. Block variables are treated the same as local scope variables in terms of naming conventions.

When declaring object or any other pointer type variables, the star `*` belongs to the variable name, not the class name or type.

Keep each variable declaration on its own line, even if subsequent declarations are of the same type.

``` objc
NSString *username;
NSString *password;
```

#### Instance Variables

Instance variables should be named like normal variables, except with a leading underscore `_`.

Do not declare them in the public `@interface` of a class, as instance variables are an implementation detail. Instead, declare them in the `@implementation` block or declare private properties in the implementation and synthesize the instance variables. See the section below for more information on interfaces and implementation.

#### Constants

Private constants should be declared with the `static` and `const` keywords and should start with the project prefix. They should **only** be declared in implementation files. `static` ensures that the symbol is only visible inside the implemention file. `const` ensures that no code in the implementation file changes the value of the symbol.

``` objc
static NSString *const JPXAnimationDurationKey = @"JPXAnimationDurationKey";
```
    
Public constants should be declared in the header and should be initialized in the implementation file. The actual value of a public constant is an implementation detail. Because of this split, the declaration must use `extern` to indicate that the symbol in the header is initialized somewhere else. The name of a public constant begins with the name of the class in which the constant is declared (in the example below `JPXProduct`), followed by a descriptive name.

``` objc
extern NSString *const JPXProductDataKey;                   // In file JPXProduct.h
NSString *const JPXProductDataKey = @"JPXProductDataKey";   // In file JPXProduct.m
```

Both private and public constants should be initialized with their symbol name. This effectively prevents conflicts with other constants.

#### Enumerations

Enumerations should be used whenever a variety of integer values could be used (usually as some kind of discriminate grouping). Enumerations should be typedef'd with the `NS_ENUM()` preprocessor macro and given a name in a similar style to their related class. The enum values should be named similarly, including the name of the type, with the actual type appended.

``` objc
typedef NS_ENUM(NSInteger, JPXProductPlacementType) {
    JPXProductPlacementTypeTop,
    JPXProductPlacementTypeCenter,
    JPXProductPlacementTypeBottom
};
```

Even though enumeration types are just integers, they should always be treated as their own type (e.g. `JPXProductPlacementType`, and never just `int`). This gives the symbol extra context instead of just some random, unrelated type. The use of `NS_ENUM()` allows the compiler to provide code-completion for enumerations and to emit more warnings if the enumeration is used inappropriately. For more information about `NS_ENUM()`, see [NSHipster - NS_ENUM & NS_OPTIONS][nshipster1]. 

You should never use a plain integer when an enumeration value is expected.

``` objc
cell.placementType = 1;                                // Bad
cell.placementType = JPXProductPlacementTypeCenter;    // Good
```

If the number the enumeration value resolves to ever changes, you get the new value for free!

#### #define

`#define` should only be used for small, internal uses where one of the above symbol types seems like overkill. `#define` is the poor-man's constant. Defines should be all uppercase, with underscores used as a delimiter. 

``` objc
#define CELL_X_OFFSET 25.0f
```

### Methods

Methods should be given descriptive names, where clarity is the most important feature. Names should be camel-cased with the initial letter lowercased.

Make method names as long as necessary, naming with hints towards the condition, cause, and return value of the method. Methods should read like sentences, and the developer should exactly grasp the purpose of the method and invocation simply by reading its name.

The single parts of the selector (i.e., the words between the colons) should be named without the use of `and`, `with`, etc. These conjunctions are not necessary and become excessive after only a few instances. Exceptions can be made for the first part of the selector.

``` objc
- (id)initWithFrame:(CGRect)frame;                                         // Good
- (id)initWithFrame:(CGRect)frame backgroundColor:(UIColor *)color;        // Good
- (id)initWithFrame:(CGRect)frame andBackgroundColor:(UIColor *)color;     // Bad
```

When creating new methods which provide more options to existing methods, list all the original parameters in your new method *first*, before adding additional parameters (as in `-initWithFrame:backgroundColor:` above). The only exception to this rule is when the final parameter of the existing method accepts a Block. In this case, the new method must keep the Block paramter as the final parameter.

``` objc
- (void)haveAPartyWithFriends:(NSArray *)friends 
               cleanUpHandler:(JPXCleanUpHandler)cleanUpHandler;
               
- (void)haveAPartyWithFriends:(NSArray *)friends 
            cateringAvailable:(BOOL)cateringAvailable 
               cleanUpHandler:(JPXCleanUpHandler)cleanUpHandler;
```

Selector signatures (both interface and implementation) must follow the standard usage of whitespace as demonstrated throughout this guide.

1. The method scope (`-` and `+` for instance or class method, respectively), followed by a space.
2. The return type of the method, with a space between the type and the star (this is really just like a cast, and all casts must follow this guideline). If there is no star, then there is no space. Double-star casts (e.g. `(NSError **)`) should not have a second space.
3. The start of the selector should directly follow the cast *without whitespace*.
4. If the method takes parameters, they should follow a colon, their cast (with no space between the colon and the cast, and no space between the cast and the argument name).
5. After the last parameter in the declaration, there should be a semicolon in the declaration.
6. After the last parameter in the implementation, there should be a space and then an opening brace **on the same line**.

``` objc
// Declaration in header
- (NSString *)stringByMashingFirstName:(NSString *)firstName lastName:(NSString *)lastName;

// Implementation in implementation file
- (NSString *)stringByMashingFirstName:(NSString *)firstName lastName:(NSString *)lastName {
    // Do something ...
}
```

Each method declaration should appear on its own line, below declared `@property` entries. Group related methods together and add a single line of whitespace between groups.

It is not necessary to declare a private method before it can be invoked from other methods. A method might also be implemented later in the class, below the call site. You should always place the method implementation where it belongs.

Finally, do not be abusive with the compiler:

* A return type must be provided, even though this is not required by the compiler.
* All parameters must be named, even though this is not required by the compiler.
* All parameters must be typed, even though this is not required by the compiler.

``` objc
- validButStupidlyNamedSelector:::;    // Bad
```

#### Functions

Always name all arguments even in the header declaration. This assists with code completion and makes it easier for the developer to know which parameters do what. Also, use the project prefix in the function name to prevent conflicting symbol names.

``` objc
void JPXPrint(NSString *);            // Bad
void JPXPrint(NSString *name);        // Mediocre
void JPXPrintName(NSString *name);    // Good
```

## Whitespace

Whitespace is free. We all have large, high resolution displays. It is good to let the code breath so it is more readable. However, when working in teams we often have to resolve merge conflicts. It is really helpful to keep the lines short to be able to read the code. Also, scrolling vertically is many times more comfortable than scrolling horizontally. 

It is recommended to hard-wrap the code after 120 characters. This gives the developer enough width to properly structure the code while gaining the ability to properly read the code without scrolling horizontally.

### General

Use spaces instead of tabs. Use a tab width of 4 spaces. Xcode will try to assist you in keeping things properly indented. Follow its suggestions (pro tip: if Xcode is being funky about how it indents things, you've probably got a syntax error somewhere). If you use tabs with a tab width of 4 spaces and your coworker uses tabs with a tab width of 8 spaces, things look messed up quickly. Using spaces instead of tabs prevents this. Also, 4 spaces in recognized to be the default in Cocoa.

If you need to wrap a method line, either the signature or an invocation, you may do so but make sure the parameter colons line up properly (Xcode will try its best to do this for you).

### Control Structures

Control structures like `if`, `for`, `while`, etc., all require one space between the keyword and the open parenthesis, and one space between the closing parenthesis and the opening brace. There should be no padding whitespace inside the parentheses before the actual condition.

``` objc
if (condition) {
    // Handle the condition
}
```

Assignments inside conditionals should be avoided, but in the case of `-init`, should be wrapped in double parentheses, as recommended by the Clang Fixit system.

Always try to use control structures without braces, as this leads to easier to understand code. If the handling of the condition requires more than a single line of code, it should be performed in a dedicated method.

``` objc
if (condition) 
    [self handleCondition];
```

## Project Conventions

### Keep the Public API Simple

The public API is what is exposed in the header files. Headers should only include the most minimal set of methods that can be used by outside classes. Internal methods should not be put in the header. This includes `@property` declarations. Public properties belong in the headers. Private properties should be declared in the **class extension** inside the implementation file.

This also means header files should not contain instance variable declarations, because they are an implementation detail. There are two other ways to declare instance variables (without dropping down to the runtime level).

1. Instance variables can be synthesized with the `@synthesize` keyword and a matching property name (and synthesized variables can even be given a custom name like so `@synthesize myPropertyName = _myPropertyName`. The propery is still `myPropertyName`, but the actual instance variable created has a leading underscore). Xcode automatically synthesizes the methods of a property and creates instance variables with a leading underscore. Therefore, explicity `@synthesize` statements should only be written when needed (e.g., when defining a lazily-initialized `readonly` property).

2. Declaring instance variables in the `@implementation`. This is just like declaring them in the `@interface`, except they are no longer visible to client classes or subclasses.

``` objc
@implementation JPXMyClass {
    NSString *_myInstanceVariable;
}
```

Of the two methods, using properties (even if just declared in the class extension) is better because it generates getters and setters, which gives us a common place to add extra code. If you access ivars directly and change must be made, then you potentially have to adjust many different call sites.

### Keep Implementations Ordered

This is an easy one. Try to keep methods in the implementation files grouped together logically instead of strewn about the file. If there are a bunch of delegate methods implemented, put those together, and use `#pragma mark - SomeDelegateProtocol` to mark the beginning of a section of methods.

For subclasses, try to keep your overrides grouped as well, in order of the hierarchy. For example, keep `NSObject` override methods first, then say, `UIViewController` overrides, then the subclass's methods.

### Class Extensions

Class extensions are how private properties should be hidden out of the public interface. They are kind of like anonymous categories. They should be placed in the `.m` file before the main `@implementation` block. For a class named `JPXShopTableViewController`, the class extension might look like this:

``` objc
@interface JPXShopTableViewController ()
    
@property (nonatomic, strong) NSString *name;

@end
```

The properties declared in the class extension are synthesized like any other property in the public interface. There is no need to create a second, named category.

### Comments

Comments in the public interface are a good thing to describe what they do, and what both the parameters and return values need to be (e.g. "This parameter must not be nil!").

For general code comments, they are often not necessary. Rather than lots of comments, the methods should be named descriptively and only do one thing. If a method needs to do several things, create more methods that are called from the first method. Comments in implementations can quickly become out of date, and are often reduntant.

Comments should always explain *why* a chunk of code exists rather than explain *what* it does. Programmers can follow along a block of code fairly easily without comments, but the real benefit is explaining something like "Shift bits to set flag to turn on networking".

#### Blocks

Use caution with Blocks, as they automatically retain objects referenced inside them. Usually this is not too much of a problem as they are also released when the Block is deallocated, but it can easily lead to retain cycles. Consider using a `__weak` local variable and in the Block to avoid a retain cycle in such cases.

#### Use Cocoa "Primitives"

Try to use the declared Cocoa "primitives" instead of their plain C counterparts. This means using `NSInteger` and `NSUInteger` instead of `int` and `unsigned int`, or using `CGFloat` instead of `float`.

We do this because it gives better flexibility if Apple ever changes what they decide an `NSInteger` should refer to. The Cocoa types are really just `typedef`'ed types that evaluate to different primitive types depending on the archictecture they are compiled for. If Apple ever switches to a 64 bit architecture for iOS, all uses of `NSInteger` will automatically be updated during the next compilation to use the proper types. We will not have to modify our code nearly as much for any possible future processor architecture changes.

It also means we should use types like `NSTimeInterval`, which really evaluates to a `double`, but using the Apple provided types are better because they give more context as to what the variable represents.

#### Exceptions

Do not use exceptions to manipulate the program flow. Throwing exceptions in Cocoa is reserved for programmer error. If there is a possibility of something going wrong at runtime, your API should require an `NSError **` (a pointer to a pointer to an NSError instance) and the method should return `NO` or `nil` if an error occurs. If that happens, the method should update the provided `NSError *` to point to a new instance of `NSError` that represents the error.

If your code crashes with an uncaught exception, there is a bug in your code and you need to fix it. A typical example is an `NSInternalInconsistencyException`. This is a mistake by the programmer, not a runtime error condition.

### ARC

All projects should use Automatic Reference Counting. Existing projects should be upgraded as soon as possible.

For projects supporting iOS 5.0 or later, you should use `weak` instead of `assign` and `strong` instead of `retain` in properties for object pointers. Keep using `assign` for primitive types, as this is exactly what should happen when setting a new value of a primitive type. Although `retain` and `strong` are synonyms in ARC, `strong` should be used instead as it gives a better and more consistent indication of what the property is doing in terms of the object graph. It is also more consistent with the storage types (like `__weak`, `__strong`, etc.).

### Warnings and Errors

Warnings should be treated as errors (at least in release builds), both by you and by the compiler. A warning is a bug that maybe has not happened yet. Fix all the warnings as soon as they happen so they do not get out of control.

Turn on the Static Analyzer for all configurations and all targets. Our machines are powerful enough to use the Static Analyzer in every build. Fix a Static Analyzer warnings as soon as it occurs.

Always aim for 0 erros, 0 compiler warnings, 0 Static Analyzer warnings.

### XIBs and Storyboards 

Interface Builder documents can be really useful and allow us to quickly prototype our interfaces, but they can usually only get us 90% of the way, with the other 10% being impossible without using code. XIBs and Storyboard documents are  difficult to merge. Due to these issues, use XIBs and Storyboards wisely and anticipate their shortcomings.

### Base SDK and Deployment Target

The project's `Base SDK` should always be set to `Latest iOS`, so it always links with the latest SDK. The Deployment Target specifies the oldest supported iOS version.

## The Boyscout Rule

Do your best to always improve the code. Leave it in a better state than how you found it. If you notice a class is messy, clean it up and make it adhere to this style guide. Never be afraid to change code for the better.

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
