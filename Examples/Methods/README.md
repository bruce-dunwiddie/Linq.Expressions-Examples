## Linq.Expressions code examples for method calls

The code examples below show the general flow of calling a method
using the System.Linq.Expressions namespace, with a few of the
common usage scenarios and explanations.

To create a delegate method (delegate, Func, or Action) using
the System.Linq.Expressions namespace, there are generally these
steps:

- Use reflection to locate the method definition as a MethodInfo instance.

- Create ParameterExpression instances to reference arguments of delegate method. This includes creating ParameterExpressions for instance parameters.

- Create MethodCallExpression instance using MethodInfo, ParameterExpressions, along with any constants or other arguments for method.

- Create generic typed Expression instance (lambda), specifying the top level method that will be the body of the lambda, along with specifying the ParameterExpressions that will correspond to the delegate arguments.

- Compile the lambda into delegate.

***

### Examples:

- [Calling simple instance method](#calling-simple-instance-method)

- [Calling simple static method](#calling-simple-static-method)

- [Calling instance method with no return](#calling-instance-method-with-no-return)

- [Pass results of method into method](#pass-results-of-method-into-method)

***

### Calling simple instance method

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

***

### Calling simple static method

This example shows how to specify calling a static method. Notice
that the instance ParameterExpression is left out when defining
the MethodCallExpression, and when defining the ParameterExpression
array of the lambda.

#### Fiddle: (https://dotnetfiddle.net/8JT99i)

#### Code:

```csharp
// locate the method to be called
MethodInfo parse = typeof(System.DateTime)
    .GetMethod(
        "Parse",
        new Type[] { typeof(string) });

// ParameterExpression is used to map a value from the caller
// to the lambda body
ParameterExpression dateAsString = Expression.Parameter(
    typeof(string));

// define the body of the lambda

// we're going to just have a single method call within the body

// we're using the overload with just two arguments because we're
// calling a static method with no instance object
MethodCallExpression body = Expression.Call(
    parse,
    new Expression[] { dateAsString });

// Func<in T, out TResult>

// first Type argument of Func definition will be Type of instance object
// second Type argument of Func definition will be Type of return

Expression<Func<string, DateTime>> lambda = Expression.Lambda<Func<string, DateTime>>(
    
    // this is going to be the function body/logic
    body,
    
    // this is the parameter list being passed in to the lambda body

    // have to use same ParameterExpression instances referenced in
    // expression body definition above to get them to map through Func call
    new ParameterExpression[] { dateAsString });

// create the Func from the lambda
Func<string, DateTime> func = lambda.Compile();

// use Func to execute method
DateTime date = func("1/2/2020 1:23:45.678");

string formattedDate = date.ToString();

Console.WriteLine(formattedDate);
```

#### Result:

```
1/2/2020 1:23:45 AM
```

***

### Calling instance method with no return

This example shows how to declare an Action delegate, which
does not provide a return value, for calling methods that
do not have a return value.

#### Fiddle: (https://dotnetfiddle.net/AY5Ymq)

#### Code:

```csharp
List<string> words = new List<string>() { "one", "two" };

// locate the method to be called
MethodInfo add = words
    .GetType()
    .GetMethod(
        "Add", 
        new Type[] { typeof(string) });

// ParameterExpression is used to map a value from the caller
// to the lambda body

// this is the same for object instances as well as method
// parameters
ParameterExpression list = Expression.Parameter(
    words.GetType());

ParameterExpression word = Expression.Parameter(
    typeof(string));

// define the body of the lambda

// we're going to just have a single method call within the body
MethodCallExpression body = Expression.Call(
    list,
    add,
    new Expression[] { word });

// Action<in T1, in T2>

Expression<Action<List<string>, string>> lambda = Expression.Lambda<Action<List<string>, string>>(
    
    // this is going to be the function body/logic

    // the value returned from a function is the last expression in its body
    body,

    // this is the parameter list being passed in to the lambda body

    // have to use same ParameterExpression instances referenced in
    // expression body definition above to get them to map through Func call

    // remember to add instance parameter
    new ParameterExpression[] { list, word });

// create the Action from the lambda
Action<List<string>, string> action = lambda.Compile();

// use Action to execute method
action(words, "three");

string wordsList = string.Join(",", words);

Console.WriteLine(wordsList);
```

#### Result:

```
one,two,three
```

***

### Pass results of method into method

This example shows how to create more complex MethodCallExpression
bodies by passing more complex expression object in as arguments
to the method, including other method calls.

#### Fiddle: (https://dotnetfiddle.net/CsLrK6)

#### Code:

```csharp
List<string> dates = new List<string>();

// locate the methods to be called
MethodInfo add = dates
    .GetType()
    .GetMethod(
        "Add",
        new Type[] { typeof(string) });

MethodInfo toString = typeof(DateTime)
    .GetMethod(
        "ToString",
        new Type[] { typeof(string) });

// ParameterExpression is used to map a value from the caller
// to the lambda body

// this is the same for object instances as well as method
// parameters
ParameterExpression list = Expression.Parameter(
    dates.GetType());

ParameterExpression date = Expression.Parameter(
    typeof(DateTime));

// use Expression.Constant to convert a 'normal' value into an expression
ConstantExpression format = Expression.Constant(
    "MMM d");

MethodCallExpression toStringCall = Expression.Call(
    date,
    toString,
    new Expression[] { format });

// define the body of the lambda

// we're passing in the body of one method call in as a
// parameter to another method call
MethodCallExpression body = Expression.Call(
    list,
    add,
    new Expression[] { toStringCall });

Expression<Action<List<string>, DateTime>> lambda = Expression.Lambda<Action<List<string>, DateTime>>(
    
    // this is going to be the function body/logic

    // the value returned from a function is the last expression in its body
    body,

    // this is the parameter list being passed in to the lambda body

    // have to use same ParameterExpression instances referenced in
    // expression body definition above to get them to map through Func call
    new ParameterExpression[] { list, date });

// create the Action from the lambda
Action<List<string>, DateTime> action = lambda.Compile();

// use Action to execute method

// this will format the date using the specified format
// and add the formatted string to the list
action(dates, DateTime.Parse("1/1/2020"));
action(dates, DateTime.Parse("1/2/2020"));
action(dates, DateTime.Parse("2/1/2020"));

string datesAsString = string.Join(",", dates);

Console.WriteLine(datesAsString);
```

#### Result:

```
Jan 1,Jan 2,Feb 1
```
