One of the most significant additions to C# 9 was records. Records help us to implement immutable types with value-based equality semantics.

Declaring a record type is pretty straightforward. First, let's declare a record type using the positional syntax.

```csharp
public record Person(string Name, string Surname, int Age);
```

Below is how the compiler rewrites when compiling the record type declared above.

```csharp
public class Person : IEquatable<Person>
{
    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private readonly string <Name>k__BackingField;

    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private readonly string <Surname>k__BackingField;

    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private readonly int <Age>k__BackingField;

    [System.Runtime.CompilerServices.Nullable(1)]
    protected virtual Type EqualityContract
    {
        [System.Runtime.CompilerServices.NullableContext(1)]
        [CompilerGenerated]
        get
        {
            return typeof(Person);
        }
    }

    public string Name
    {
        [CompilerGenerated]
        get
        {
            return <Name>k__BackingField;
        }
        [CompilerGenerated]
        init
        {
            <Name>k__BackingField = value;
        }
    }

    public string Surname
    {
        [CompilerGenerated]
        get
        {
            return <Surname>k__BackingField;
        }
        [CompilerGenerated]
        init
        {
            <Surname>k__BackingField = value;
        }
    }

    public int Age
    {
        [CompilerGenerated]
        get
        {
            return <Age>k__BackingField;
        }
        [CompilerGenerated]
        init
        {
            <Age>k__BackingField = value;
        }
    }

    public Person(string Name, string Surname, int Age)
    {
        <Name>k__BackingField = Name;
        <Surname>k__BackingField = Surname;
        <Age>k__BackingField = Age;
        base..ctor();
    }

    [System.Runtime.CompilerServices.NullableContext(1)]
    public override string ToString()
    {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.Append("Person");
        stringBuilder.Append(" { ");
        if (PrintMembers(stringBuilder))
        {
            stringBuilder.Append(' ');
        }
        stringBuilder.Append('}');
        return stringBuilder.ToString();
    }

    [System.Runtime.CompilerServices.NullableContext(1)]
    protected virtual bool PrintMembers(StringBuilder builder)
    {
        RuntimeHelpers.EnsureSufficientExecutionStack();
        builder.Append("Name = ");
        builder.Append((object)Name);
        builder.Append(", Surname = ");
        builder.Append((object)Surname);
        builder.Append(", Age = ");
        builder.Append(Age.ToString());
        return true;
    }

    [System.Runtime.CompilerServices.NullableContext(2)]
    public static bool operator !=(Person left, Person right)
    {
        return !(left == right);
    }

    [System.Runtime.CompilerServices.NullableContext(2)]
    public static bool operator ==(Person left, Person right)
    {
        return (object)left == right || ((object)left != null && left.Equals(right));
    }

    public override int GetHashCode()
    {
        return ((EqualityComparer<Type>.Default.GetHashCode(EqualityContract) * -1521134295 + EqualityComparer<string>.Default.GetHashCode(<Name>k__BackingField)) * -1521134295 + EqualityComparer<string>.Default.GetHashCode(<Surname>k__BackingField)) * -1521134295 + EqualityComparer<int>.Default.GetHashCode(<Age>k__BackingField);
    }

    [System.Runtime.CompilerServices.NullableContext(2)]
    public override bool Equals(object obj)
    {
        return Equals(obj as Person);
    }

    [System.Runtime.CompilerServices.NullableContext(2)]
    public virtual bool Equals(Person other)
    {
        return (object)this == other || ((object)other != null && EqualityContract == other.EqualityContract && EqualityComparer<string>.Default.Equals(<Name>k__BackingField, other.<Name>k__BackingField) && EqualityComparer<string>.Default.Equals(<Surname>k__BackingField, other.<Surname>k__BackingField) && EqualityComparer<int>.Default.Equals(<Age>k__BackingField, other.<Age>k__BackingField));
    }

    [System.Runtime.CompilerServices.NullableContext(1)]
    public virtual Person <Clone>$()
    {
        return new Person(this);
    }

    protected Person([System.Runtime.CompilerServices.Nullable(1)] Person original)
    {
        <Name>k__BackingField = original.<Name>k__BackingField;
        <Surname>k__BackingField = original.<Surname>k__BackingField;
        <Age>k__BackingField = original.<Age>k__BackingField;
    }

    public void Deconstruct(out string Name, out string Surname, out int Age)
    {
        Name = this.Name;
        Surname = this.Surname;
        Age = this.Age;
    }
}
```
In C# 9, records were reference types(`class`). In addition to record classes, C# 10 brings record structs and allows us to declare and use value type records(`struct`). Declaring a record struct is pretty similar to declaring a record class.

```csharp
public record struct Person(string Name, string Surname, int Age);
```

Since you can declare record structs using the `record struct` keywords, you can also use `record class` in addition to `record` to declare record classes in C# 10.

```csharp
//Record classes
public record Person(string Name, string Surname, int Age);
public record class Person(string Name, string Surname, int Age);

//Record structs
public record struct Person(string name, string surname, int age);
```

Let's have a look at the code compiler generates.

```csharp
public struct Person : IEquatable<Person>
{
    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private string <Name>k__BackingField;

    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private string <Surname>k__BackingField;

    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private int <Age>k__BackingField;

    public string Name
    {
        [IsReadOnly]
        [CompilerGenerated]
        get
        {
            return <Name>k__BackingField;
        }
        [CompilerGenerated]
        set
        {
            <Name>k__BackingField = value;
        }
    }

    public string Surname
    {
        [IsReadOnly]
        [CompilerGenerated]
        get
        {
            return <Surname>k__BackingField;
        }
        [CompilerGenerated]
        set
        {
            <Surname>k__BackingField = value;
        }
    }

    public int Age
    {
        [IsReadOnly]
        [CompilerGenerated]
        get
        {
            return <Age>k__BackingField;
        }
        [CompilerGenerated]
        set
        {
            <Age>k__BackingField = value;
        }
    }

    public Person(string Name, string Surname, int Age)
    {
        <Name>k__BackingField = Name;
        <Surname>k__BackingField = Surname;
        <Age>k__BackingField = Age;
    }

    [IsReadOnly]
    public override string ToString()
    {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.Append("Person");
        stringBuilder.Append(" { ");
        if (PrintMembers(stringBuilder))
        {
            stringBuilder.Append(' ');
        }
        stringBuilder.Append('}');
        return stringBuilder.ToString();
    }

    [IsReadOnly]
    private bool PrintMembers(StringBuilder builder)
    {
        builder.Append("Name = ");
        builder.Append((object)Name);
        builder.Append(", Surname = ");
        builder.Append((object)Surname);
        builder.Append(", Age = ");
        builder.Append(Age.ToString());
        return true;
    }

    public static bool operator !=(Person left, Person right)
    {
        return !(left == right);
    }

    public static bool operator ==(Person left, Person right)
    {
        return left.Equals(right);
    }

    [IsReadOnly]
    public override int GetHashCode()
    {
        return (EqualityComparer<string>.Default.GetHashCode(<Name>k__BackingField) * -1521134295 + EqualityComparer<string>.Default.GetHashCode(<Surname>k__BackingField)) * -1521134295 + EqualityComparer<int>.Default.GetHashCode(<Age>k__BackingField);
    }

    [IsReadOnly]
    public override bool Equals(object obj)
    {
        return obj is Person && Equals((Person)obj);
    }

    [IsReadOnly]
    public bool Equals(Person other)
    {
        return EqualityComparer<string>.Default.Equals(<Name>k__BackingField, other.<Name>k__BackingField) && EqualityComparer<string>.Default.Equals(<Surname>k__BackingField, other.<Surname>k__BackingField) && EqualityComparer<int>.Default.Equals(<Age>k__BackingField, other.<Age>k__BackingField);
    }

    [IsReadOnly]
    public void Deconstruct(out string Name, out string Surname, out int Age)
    {
        Name = this.Name;
        Surname = this.Surname;
        Age = this.Age;
    }
}
```

Most of the generated code looks similar to the record classes. However, there is one difference between record structs and record classes. **The generated properties in record structs are mutable by default.** The reason behind this decision was to be consistent with tuples. Tuples are like anonymous record structs with similar features.
On the other hand, struct mutability does not carry the same level of concern as class mutability does. That's why the C# team decided to make record struct properties mutable by default. However, if you need immutable record structs, you can use the `readonly` keyword. 

```csharp
public readonly record struct Person(string name, string surname, int age);
```

Generated code

```csharp
public struct Person : IEquatable<Person>
{
    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private string <name>k__BackingField;

    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private string <surname>k__BackingField;

    [CompilerGenerated]
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    private int <age>k__BackingField;

    public string name
    {
        [IsReadOnly]
        [CompilerGenerated]
        get
        {
            return <name>k__BackingField;
        }
        [CompilerGenerated]
        set
        {
            <name>k__BackingField = value;
        }
    }

    public string surname
    {
        [IsReadOnly]
        [CompilerGenerated]
        get
        {
            return <surname>k__BackingField;
        }
        [CompilerGenerated]
        set
        {
            <surname>k__BackingField = value;
        }
    }

    public int age
    {
        [IsReadOnly]
        [CompilerGenerated]
        get
        {
            return <age>k__BackingField;
        }
        [CompilerGenerated]
        set
        {
            <age>k__BackingField = value;
        }
    }

    public Person(string name, string surname, int age)
    {
        <name>k__BackingField = name;
        <surname>k__BackingField = surname;
        <age>k__BackingField = age;
    }

    [IsReadOnly]
    public override string ToString()
    {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.Append("Person");
        stringBuilder.Append(" { ");
        if (PrintMembers(stringBuilder))
        {
            stringBuilder.Append(' ');
        }
        stringBuilder.Append('}');
        return stringBuilder.ToString();
    }

    [IsReadOnly]
    private bool PrintMembers(StringBuilder builder)
    {
        builder.Append("name = ");
        builder.Append((object)name);
        builder.Append(", surname = ");
        builder.Append((object)surname);
        builder.Append(", age = ");
        builder.Append(age.ToString());
        return true;
    }

    public static bool operator !=(Person left, Person right)
    {
        return !(left == right);
    }

    public static bool operator ==(Person left, Person right)
    {
        return left.Equals(right);
    }

    [IsReadOnly]
    public override int GetHashCode()
    {
        return (EqualityComparer<string>.Default.GetHashCode(<name>k__BackingField) * -1521134295 + EqualityComparer<string>.Default.GetHashCode(<surname>k__BackingField)) * -1521134295 + EqualityComparer<int>.Default.GetHashCode(<age>k__BackingField);
    }

    [IsReadOnly]
    public override bool Equals(object obj)
    {
        return obj is Person && Equals((Person)obj);
    }

    [IsReadOnly]
    public bool Equals(Person other)
    {
        return EqualityComparer<string>.Default.Equals(<name>k__BackingField, other.<name>k__BackingField) && EqualityComparer<string>.Default.Equals(<surname>k__BackingField, other.<surname>k__BackingField) && EqualityComparer<int>.Default.Equals(<age>k__BackingField, other.<age>k__BackingField);
    }

    [IsReadOnly]
    public void Deconstruct(out string name, out string surname, out int age)
    {
        name = this.name;
        surname = this.surname;
        age = this.age;
    }
}
```

## with Expressions

Record structs support `with` expressions to create copies. In C# 10, we can use the `with` expressions with all struct types including tuples.

```csharp
//with expressions - record structs
var p = new Person("Ilkay", "Ilknur", 33);
var newP = p with { Age = 50 };

public record struct Person(string Name, string Surname, int Age);

// with expressions - regular structs
var pStruct = new PersonStruct()
{
    Name = "Ilkay"
};

var pNewStruct = pStruct with { Name = "John" };

public struct PersonStruct
{
    public string Name;
}
```

## Deconstruction

Positional record structs support deconstruction similar to positional record classes. 

```csharp
var p = new Person("Ilkay", "Ilknur", 33);
var (name, surname, age) = p;

public record struct Person(string Name, string Surname, int Age);
```

## ToString Method

The synthesized `ToString` method prints the type name and values.  

```csharp
var p = new Person("Ilkay", "Ilknur", 33);
Console.WriteLine(p);
// prints : Person { Name = Ilkay, Surname = Ilknur, Age = 33 }

public record struct Person(string Name, string Surname, int Age);
```

Record structs are regular structs, and all struct rules apply to record structs as well. However, there are some additional limitations and rules we need to keep in mind.

* Record structs cannot use `ref` modifier.
* Record struct parameters cannot use `ref`, `out` or `this` modifiers (but `in` and `params` are allowed).
* Record structs inherit from `ValueTuple` implicitly.

