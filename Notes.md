# **Clean Coding Principles in C#**
> _Understanding the original programmer's intent is the biggest problem_
>
> Think like an author - use headings, paragraphs, etc.

___
## **1. Naming**
``` C#
// Good
List<decimal> prices = new List<decimal>() {5.5m, 1.48m};
decimal total = 0;
foreach (var price in prices)
{
    total += price;
}
return total;
```
The only thing that changes here is the naming of variables
``` C#
// Bad
List<decimal> p = new List<decimal>() {5.50m, 1.48m};
decimal t = 0;
foreach(var i in p)
{
    t += i;
}
return t;
```

### Verbalize / Rubber Ducking
- If you're having trouble naming a class or method, verbalizing aids creativity
- Try talking to a rubber duck

### Naming Classes
- Noun
- Be specific: _specific names encourage small, cohesive classes_
- Single Responsibility  
    - The `ProductRepository` class shouldn't send emails. The name can help prevent unnecessary logic inside the class.
- Avoid generic suffixes  
    - (Product class should not be named `ProductManager`, or `ProductInfo`). Ask whether an instance of a class is logical.

### Naming Methods
- Note that most lifecyle methods are poorly named. To offset this, you can place only method calls inside lifecycle methods to increase readability
- Select a name so descriptive, that the reader doesn't need to read the method to know what it does
    - Good examples include `GetRegisteredUsers`, `IsValidSubmission`, `ImportDocument`, `SendEmail`
    - Bad examples include `Go`, `Complete`, `Get`, `Process`, `DoIt`, `Start`, `On_Init`, `Page_Load`
- Avoid side effects  
    - CheckPassword shouldn't log users out, and ValidateSubmission shouldn't save
    - Method names should be comprehensive and tell the whole truth
- Avoid and/if/or in your method names
- Avoid abbreviations
    - Few words have standard, unambiguous abbreviations
    - Awkwardly abbreviated names hinder discussion of our code

### Naming Variables
- Booleans should be named as if asking a question
    - Good examples include `isOpen`, `done`, `isActive`, `loggedIn`
    - Bad examples include `open`, `start`, `status`, `login`
    - think: `if (done)` vs `if (start)`
- Opposing variables should be named with symmetry
    - Good: `on`/`off`, `fast`/`slow`, `lock`/`unlock`, `min`/`max`
    - Bad: `on`/`disable`, `quick`/`slow`, `lock`/`open`, `slow`/`max`

___
## **2. Conditionals**

### Compare Booleans Implicitly
- `if (loggedIn)` instead of `if (loggedIn == true)`

### Assign Booleans Implicitly
``` C#
// Good
bool goingToChipotleForLunch = cashInWallet > 6.00;
```
``` C#
// Bad
bool goingToChipotleForLunch;
if (cashInWallet > 6.00)
{
    goingToChipotleForLunch = true;
}
else
{
    goingToChipotleForLunch = false;
}
```
- If the code sounds funny when spoken out loud, it's probably funny when read as well.

### Be Positive
- Don't use double negatives:
    - `if (loggedIn)` instead of `if (!isNotLoggedIn)`
    - Negative conditionals are often a result of a sign of programming by accident

### Ternary is Beautiful 
*Note: Don't nest ternaries
``` C#
// Better
int registrationFee = isSpeaker ? 0 : 50;
```
``` C#
// Good
int registrationFee;
if (isSpeaker)
{
    registrationFee = 0;
}
else
{
    registrationFee = 50;
}
```

### Be Strongly Typed (not stringly typed)
- `employee.type == EmployeeType.Manager` instead of `employeeType == "manager"`
- Leverage enums instead of strings
    - No typos resulting in runtime failure
    - You get Intellisense support
    - You can easily find all possible values
    - Enums are more searchable

### Avoid Magic Numbers
- Doing this avoids typos, gains searchability, and clarifies intent
``` C#
// Good
const int legalDrinkingAge = 21;
if (age > legalDrinkingAge) 
{
}

// Also Good
if (status == Status.active) 
{
}
```
``` C#
// Bad
if (age > 21) 
{
}

// Still Bad
if (status == 2) 
{
}
```

### Handling Complex Conditionals
1. Use intermediate variables
    ``` C#
    bool eligibleForPension = employee.Age > 55
        && employee.YearsEmployed > 10
        && employee.IsRetired == true;

    if (eligibleForPension) { }
    ```
    ``` C#
    // Bad
    // What question is this actually answering?
    if (employee.Age > 55
        && employee.YearsEmployed > 10
        && employee.IsRetired == true
    {
        // Do lots of things here
    }
    ```
2. Encapsulate via function: Favor expressive code over comments
    ``` C#
    // Better - it's longer, but clarifies intent, and reads better when verbalized
    private bool ValidFileRequest(string fileExtension, bool isActiveFile, bool isAdmin)
    {
        var validFileExtensions = new List<string>() { "mp4", "mpg", "avi" };

        bool validFileType = validFileExtensions.Contains(fileExtension);
        bool userIsAllowedToViewFile = isActiveFile || isAdmin;

        return validFileType && userIsAllowedToViewFile
    }
    ```
    ``` C#
    // Good
    private bool ValidFileRequest(string fileExtension, bool isActiveFile, bool isAdmin)
    {
        return ((fileExt == ".mp4"
            || fileExt == ".mpg"
            || fileExt = ".avi")
            && (sAdmin == 1 || isActiveFile))
    }
    ```
    ``` C#
    // Bad
    // Check for valid file extensions, confirm is admin or active
    if ((fileExt == ".mp4"
            || fileExt == ".mpg"
            || fileExt = ".avi")
            && (sAdmin == 1 || isActiveFile))
    ```

### Favor Polymorphism Over Switch
``` C#
// Good - Each class handles itself through polymorphism
// In general, there would still be a switch statement within a factory,
// determining which of the classes would be instantiated
public void LoginUser(User user)
{
    user.Login()
}

public abstract class User
{
    public string FirstName;
    public string LastName;
    public int Status;
    public int AccountBalance;

    public abstract void Login();
}

public class ActiveUser : User
{
    public override void Login()
    {
        // Active user logic here
    }
}

public class InactiveUser : User
{
    public override void Login()
    {
        // Inactive user logic here
    }
}
```

``` C#
// Bad
public void LoginUser(User user)
{
    switch (user.Status)
    {
        case Status.Active:
            // active user logic
            break;
        case Status.Inactive:
            // inactive user logic
            break;
        case Status.Locked:
            // locked user logic
            break;
    }
}
```

### Be Declarative
``` C#
// Good
return users
    .Where(u => u.AccountBalance < minAccountBalance>)
    .Where(u => u.Status == Status.Active);
```

``` C#
// Bad
List<User> matchingUsers = new List<User>();

foreach (var user in users)
{
    if (user.AccountBalance < minAccountBalance && user.Status == Status.Active)
    {
        matchingUsers.Add(user)
    }
    return matchingUsers;
}
```

### Table Driven Methods
> Sometimes code isn't the answer

Examples
- Insurance rates
- Pricing structures
- Complex and dynamic business rules

Table driven methods are
- Great for dynamic logic
- Avoids hard coding
- Write less code
- Avoids complex data structures
- Make changes without a code deployment
``` C#
// Good
return Repository.GetInsuranceRate(age);
```

``` C#
// This kind of logic belongs in a database
if (age < 20)
{
    return 345.60m; // avoid hard coding values that change
}
else if (age < 30)
{
    return 419.50m;
}
else if (age < 40)
{
    return 476.38m;
}
else if (age < 50)
{
    return 516.25m;
}
```
___
## **3. Writing Clean Methods**

### When to Create a Function or Method
1. Avoid duplication - Don't Repeat Yourself
2. Indentation (less)
3. Clarify intent, just like headings in a book
4. Do just one thing

### Duplication
- Key: Don't repeat yourself. Code is a liability
- Look for patterns in your code

### Indentation
- Deep indentation is hard to read.
    - A sign of *high cyclomatic complexity* (a measure of the number of paths through the code)
    - hinders testing
    - increases vulnerability to bugs
    - *Comprehension decreases beyond three levels of nested "if" blocks*
- #### Extract Method
    - Like a footnote, or the links you see in Wikipedia to other articles
    ``` C#
    // AFter
    if
        if
            doComplicatedThing()
        end if
    end if

    doComplicatedThing()
    {
        while
            // do some complicated thing
        end while
    }
    ```
    ``` C#
    // Before
    if
        if
            while
                // do some complicated thing
            end while
        end if
    end if
    ```
- #### Fail Fast and Return Early
    - Throw exceptions in a switch's default statement
    - Use guard clauses that return or throw exceptions, instead of nesting ifs
    ``` C#
    // Uses guard clauses
    public void RegisterUser(string username, string password)
    {
        if (!string.IsNullOrWhiteSpace(username))
        {
            throw new ArgumentException("Username is required.");
        }
        if (!string.IsNullOrWhiteSpace(password))
        {
            throw new ArgumentException("Password is required.");
        }
        // register user
    }
    ```
    - if you have nothing more to do; return!
    ``` C#
    // Multiple returns aren't always bad
    private bool ValidUserName(string userName)
    {
        const int MinUserNameLength = 6;
        if (userName.Length < MinUserNameLength)
        {
            return false;
        }

        const int MaxUserNameLength = 25;
        if (userName.Length > MaxUserNameLength)
        {
            return false;
        }
        // and so on
    }
    ```
    ``` C#
    // Bad
    public void RegisterUser(string username, string password)
    {
        if (!string.IsNullOrWhitespace(username))
        {
            if (!string.IsNullOrWhitespace(password))
            {
                // register user
            }
            else
            {
                throw new ArgumentException("Password is required");
            }
        }
        else
        {
            throw new ArgumentException("Username is required")
        }
    }
    ```
    ``` C#
    // Way too much nesting
    private bool ValidUserName(string userName)
    {
        bool isValid is false;
        const int MinUserNameLength = 6;
        if (userName.Length >= MinUserNameLength)
        {
            const int MaxuserNameLength = 25;
            if (userName.Length <= MaxUserNameLength>)
            {
                // even more nesting, to set isValid to true
            }
        }
        return isValid
    }
    ```
- #### Convey Intent
    ``` C#
    private bool ValidFileRequest(string fileExtension, bool isActiveFile, bool isAdmin)
    {
        // tells you if it's a valid file request
    }
    ```
    ``` C#
    // Check for valid file extensions, confirm is admin or active
    if ((fileExt == ".mp4" || fileExt == ".mpg" || fileExt == ".avi") 
        && (isAdmin == 1 || isActiveFile))
    ```
- #### Do One Thing
    > Would you read a book without paragraphs?
    - Aids the reader
    - promotes reuse
    - Eases naming and testing
    - Avoids side-effects
        - We should never be surprised at what a method does

### Mayfly Variables - only live a few hours!
- Initializing all your variables together up top actually breaks the rule of 7
- Minimize the number of variables in scope
- Initialize variables just-in-time. Bring it to life as needed, and break their scope quickly

### How Many Parameters?
- Strive for 0-2 parameters
    - Easier to read, understand, and test
    - Ensures the method is doing only one thing
    - Booleans are often known as flag arguments, and are a strong sign the function is doing more than one thing
    ``` C#
    // Emails, printing, and billing are separate concerns from saving a user.
    public void SaveUser(User user, bool sendEmail, int emailFormat, bool printReport, bool sendBill)
    ```

### Signs It's Too Long
- Whitespace & comments
- Scrolling is required
- Hard to name - a single, clear defined task is easy to name
- Multiple conditionals - consider calling a separate function to extract out the conditional
- Hard to digest
    - Too many layers of abstraction
    - Remember the rule of 7
- Robert Martin:
    - Rarely over 20 lines, hard limit of 100
    - Rarely over 3 parameters

**Simple functions can be longer. Complex functions should be short.**  
From the Linux style guide:
> The maximum length ... is inversely proportional to the complexity and indentation level of
> that function. So, if you have a conceptually simple function that is just one long 
> (but >simple) case statement ... It's OK to have a longer function ... if you have 
> a complex function ... adhere to limits all the more closely

### Handling Exceptions
Only catch exceptions you can handle. Avoid catching unrecoverable exceptions
Can't handle the exception? Fail fast. Fail loud.  
Let a broken app crash. Don't just let it limp along.  
Throw an unchecked exception.
#### - Unrecoverable
- e.g. null reference, file not found, or access denied
- The most common exception type
#### - Recoverable
- e.g. retry connection, try different file, wait and try again
- Important to consider giving up at some point
#### - Ignorable
- e.g. Logging click
- Very rare

``` C#
// More readable
try
{
    SaveThePlanet();
}
catch (Exception e)
{
    // do something
}

private void SaveThePlanet()
{
    // many lines
    // of complicated
    // logic here
}
```
``` C#
// Should have moved to a function
try
{
    // many lines
    // of complicated
    // logic here
}
catch (Exception e)
{
    // do something here
}
```
``` C#
// Try/Catch/Log = Fail Slow
// Don't just log exceptions, unless they're really ignorable!
try
{
    RegisterSpeaker();
}
catch (Exception e)
{
    LogError(e)
}
EmailSpeaker(); // This will email the speaker even if registration fails!
```

___
## **4. Classes**
Classes are like book headings. Should contain multiple related methods

### When to Create a Class
- To model an object
- If your methods are only loosely related, you should split them into separate classes
- To promote reuse
- Reduce complexity - Solve once, hide away
- Clarify parameters - If you have a lot of related variables, you can use a class

### Class Cohesion Overiew
Cohesion is how strongly the responsibilities of a class are related.
Each module should do one thing well, and have a single reason to change.
- Enhances readability
- Increases likelihood of reuse - the logic inside is more likely to be discovered
- Avoids attracting the lazy - Magnet classes (broken window effect)
    - Keep class names descriptive
- #### Watch out for:
    - Standalone methods
    - Fields used by only one method
        - fields should be used by many methods
        - if the field is not used by more than one method, it should maybe be extracted to a separate class
    - Classes that change often
    - Classes with too many dependencies (why need so many if you have just one job?)

### Low vs High Cohesion
Cohesion becomes increasingly important as classes grow
- Measure cohesion by how many methods use the class' instance variables

High Cohesion:
- Vehicle
    - Edit vehicle options
    - update pricing
- VehicleMaintenance
    - Schedule maintenance
    - Send maintenance reminder
- VehicleFinance
    - Select financing
    - Calculate monthly payment

Low Cohesion: Many unrelated methods
- Edit vehicle options
- Update pricing
- Schedule maintenance
- Send maintenance reminder
- Select financing
- Calculate monthly payment

### Names and Cohesion
- Broad names lead to poor cohesion and magnet classes
- Use specific names to fend off lazy developers
- Use Nouns

### Signs a Class is Too Small
- If two classes rely heavily on one another, they may need to be a single class
- Too many pieces in a system
- These issues are all extremely rare

### Primitive Obsession
If a method uses only one or two properties, consider passing in only those. However, in general, don't pass a bunch of primitives, group related items in a class
``` C#
// Good
private void SaveUser(User user)
```
``` C#
// Too many primitives
private void SaveUser(string firstName, string lastName, string state, string zip, string eyeColor, string phone, string fax, string maidenName)
```

### Proximity Principle
Make code read top to bottom when possible. Keep related actions together.

### The Outline Rule
Consider creating multiple layers of abstraction
Collapsed code should read like an outline, e.g:
- Class can be roughly read as Method 1, 2, 3. You could drill down from there
    - Method 1
        - Method 1a
            - Method 1ai
            - Method 1aii
        - Method 1b
            - Method 1bi
            - Method 1bii
        - Method 1c
    - Method 2
        - Method 2a
        - Method 2b
    - Method 3
        - Method 3a
        - Method 3b

- Bad Class is more obscure to read
    - Method 1
        - Method 1a
            - Method 1ai
            - Method 1aii
            - Method 1aiii
        - Method 1b
        - Method 1c

___
## **5. Writing Clean Comments**
An over-reliance on comments is a code smell. Use comments only when there are no other alternatives.

### Comments: A Necessity and a Crutch
- Prefer expressive code over comments. Code is more likely than comments to be kept updated
- Use comments when code isn't sufficient

### Comments to Avoid:
- #### Redundant Comments
    1. Assume your reader can read code
    2. Don't repeat yourself. (Repeating your code through comments)
    ``` C#
    int i = 1; // Set i = 1
    ```
    ``` C#
    var cory = new User(); // Instantiate a new user
    ```
    ``` C#
    /// <summary>
    /// Default Constructor
    /// </summary>
    public User() { }
    ```
    Avoid requiring comments for every method
    ``` C#
    /// <summary>
    /// Calculates Total Charges
    /// </summary>
    private void CalculateTotalCharges()
    {
        // Total charges calculated here
    }
    ```
- #### Intent Comments
    You can eliminate intent comments via
    - A well-named constant or enum
    - Improved function name
    - Declaring intermediate variable
    - Extract conditional to function

    ``` C#
    if (user.Status == Status.Inactive)
    ```
    ``` C#
    // Assure user's account is deactivated
    if (user.Status == 2)
    ```
- #### Apology Comments
    Don't apologize
    - Fix it before commit/merge
    - Add a TODO marker comment if you must
    ``` C#
    // Sorry, this crashes sometimes so I'm swallowing the exception. Lol.
    ```
    ``` C#
    // I was too tired to refactor this pile when I was done...
    ```
- #### Warning Comments
    Warning comments often declare that a section of code "belongs" to someone, and is a sign that refactoring is necessary.
- #### Zombie Code
    Large sections of commented-out code kills readability. It effects searches and refactoring.  
    Code is a liability. Less is more
- #### Divider Comments
    ``` C#
    // Instead of using comments to section off methods, this should have been broken up into more methods
    private void MyLongFunction()
    {
        // lots
        // of
        // code

        // Start search for available concert tickets
        
        // lots
        // of
        // concert
        // search
        // code

        // End of concert ticket search

        // lots
        // more
        // code
    }
    ```
- #### Brace Tracker Comments
    ``` C#
    // Abstraction helps readability and decreases cyclomatic complexity
    private void AuthenticateUsers()
    {
        bool validLogin = false;
        // deeply
            // nested
                // code
                if (validLogin)
                {
                    LoginUser();
                }
            // even
        // more code
    }
    ```
    ``` C#
    // Bad
    private void AuthenticateUsers()
    {
        bool validLogin = false;
        // deeply
            // nested
                // code
                if (validLogin)
                {
                    // lots
                    // of
                    // code
                    // here
                } // end user login <-- brace tracker was used to mark the end of a long block
            // even
        // more code
    }
    ```
- #### Bloated Headers
    - Avoid line endings. The asterisks in the example below are so tedious to maintain that devs may just end up not updating the header
    - Don't repeat yourself. e.g. Filenames are obviously accesible by viewing the file
    - Follow your language's conventions for headers
    ``` C#
    //***************************************************
    // Filename: Monolith.cs                            *
    // Author: Ben Wong                                 *
    // Created: 02/09/2020                              *
    // Weather that day: Snow                           *
    //                                                  *
    // Summary                                          *
    // This class does a great many things. To make it  *
    // extra useful I placed pretty much all the app    *
    // logic here. You wish your class was this         *
    // powerful. Bwahhahha!                             *
    //***************************************************
    ```
- #### Defect Log
    Change metadata belongs in source control, not the comments. A well written book isn't covered in author notes.
    ``` C#
    // Defect #5274 DA 12/10/2010
    // We weren't checking for null here
    if (firstName != null)
    {
        //....
    }
    ```
- #### Clean Comments
    ##### To do comments - Watch out, because these can be abused, or ignored
    - Standardize with your team
    - Ensure they aren't a warning or apology in disguise
    ``` C#
    // TODO Refactor out duplication
    ```
    ``` C#
    // HACK The API doesn't expose needed call yet.
    ```
    ##### Summary Comments
    - Describes intent at general level higher than the code. Sometimes a class name alone isn't high level enough. Don't use to simply augment poor naming.
    ``` C#
    // Encapsultes logic for calculating retiree benefits
    ```
    ``` C#
    // Generates custom newsletter emails
    ```
    ##### Documentation
    - Useful when it can't be expressed in code.
    ``` C#
    // See microsoft.com/api for documentation
    ```

___
## 6. Staying Clean

### When to Refactor
- Refactor when you need to work with the code.
    > Refactoring for future readers is of questionable value, since it means that you're investing time now for readers that may not materialize in the future
    > If code is hard to read, but it exists in a system that's been chugging along reliably for years, then don't risk changing it merely in a desire for cleanliness
- Refactor when the code is hard for you to difficult to comprehend or change.
- In general, the size of the refactor should be proportional to the size of the code you need to change.
- Add tests for regression protection before you refactor.

### Code Reviews
- Promote proactive cleanliness
- Set guidelines for the team.
- Assure readability
- Working in pairs is like real-time code review
    - Increases quality
    - Naming and refactoring is easier

### Lunch and Learns
- Showcase what you've learned
- Be a thought leader
- Spark interest in others
- Free food

### Accept No Broken Windows
A neglected codebase will only become more neglected

### Boy Scout Rule
Leave the code you're editing a little better than you found it.  
    ~ Robert Martin