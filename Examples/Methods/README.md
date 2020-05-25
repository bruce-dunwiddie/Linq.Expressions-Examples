[Calling simple instance method](#calling-simple-instance-method)

## Calling simple instance method

### Code:

```csharp
DateTime testDate = DateTime.Parse("1/2/2020 1:23:45.678");

// locate the method to be called
MethodInfo toString = testDate
    .GetType()
    .GetMethod(
        "ToString",
        new Type[] { typeof(string) });

ParameterExpression date = Expression.Parameter(
    testDate.GetType());

ConstantExpression format = Expression.Constant(
    "hh:mm:ss");			

MethodCallExpression body = Expression.Call(
    date,
    toString,
    new Expression[] { format });

// Func<in T, out TResult>

// first Type argument of Func definition will be Type of instance object
// second Type argument of Func definition will be Type of return

Expression<Func<DateTime, string>> lambda = Expression.Lambda<Func<DateTime, string>>(

    // this is going to be the function body/logic
    body,

    // these is the parameter list being passed in to the lambda body

    // have to use same ParameterExpression instances referenced in
    // expression body definition above to get them to map through Func call
    new ParameterExpression[] { date });

Func<DateTime, string> func = lambda.Compile();

// use Func to execute method
string formattedDate = func(testDate);

Console.WriteLine(formattedDate);
```
### Result:
```
01:23:45
```