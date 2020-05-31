Code examples of System.Linq.Expressions namespace class usage with explanations.

### Topics

- [Methods](Examples/Methods/)

- Constructors (Coming Soon)

- Fields (Coming Soon)

- [Properties](Examples/Properties/)

- Blocks (Coming Soon)

- Variables (Coming Soon)

- Constants (Coming Soon)

- Casts (Coming Soon)

- Loops (Coming Soon)

- Conditions (Coming Soon)

Complete unit tests with compilable project can also be found [here](https://github.com/bruce-dunwiddie/Linq.Expressions-Examples-Code).

### Sample Code Example for Calling Instance Method

This example shows the general flow of creating the lambda delegate for
executing an instance method. 

#### Fiddle: (https://dotnetfiddle.net/hFssOH)

#### Code:

```csharp
DateTime testDate = DateTime.Parse("1/2/2020 1:23:45.678");

// locate the method to be called
MethodInfo toString = testDate
    .GetType()
    .GetMethod(
        "ToString",
        new Type[] { typeof(string) });

// ParameterExpression is used to map a value from the caller
// to the lambda body
ParameterExpression date = Expression.Parameter(
    testDate.GetType());

// use Expression.Constant to convert a 'normal' value into an expression
ConstantExpression format = Expression.Constant(
    "hh:mm:ss");			

// define the body of the lambda

// we're going to just have a single method call within the body
MethodCallExpression body = Expression.Call(
    date,
    toString,
    new Expression[] { format });

// Func<in T, out TResult>

// first Type argument of Func definition will be Type of instance object
// second Type argument of Func definition will be Type of return

Expression<Func<DateTime, string>> lambda = Expression.Lambda<Func<DateTime, string>>(

    // this is going to be the function body/logic
    
    // the value returned from a function is the last expression in its body
    body,

    // this is the parameter list being passed in to the lambda body

    // have to use same ParameterExpression instances referenced in
    // expression body definition above to get them to map through Func call
    new ParameterExpression[] { date });

// create the Func from the lambda
Func<DateTime, string> func = lambda.Compile();

// can now use Func to execute instance method on object instance
// and get result
string formattedDate = func(testDate);

Console.WriteLine(formattedDate);
```

#### Result:

```
01:23:45
```
