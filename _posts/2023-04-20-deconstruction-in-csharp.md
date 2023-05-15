---
layout: post
title: Deconstruction in C#
date: 2023-04-20 11:45 +0300
categories: [csharp]
tags: [dotnet] 
---
In the past, when I needed to retrieve multiple values from a method, I would typically resort to either creating a new class or using an out parameter, which always felt a bit cumbersome. However, while exploring the features of c# 7, I stumbled upon something that I had been missing all along - Tuple literals and deconstruction. This discovery was a real gem, as it provided exactly what I had been looking for. So, let's delve a little deeper into what this is all about.

## Tuple deconstruction

Tuple literal is a shorthand syntax for creating a tuple instance with specific values.

```cs
var myTuple = (33, "Hi!", true);
```

Let's see how we could use this in a method. Imagine that you want to retrieve some data for a person such as, the first name, the last name and the age.
```cs
(string,string,int) GetPerson(Guid id)
{
    // Get data from the Database
    // for the sake of the example lets hardcode the values
    string firstName = "John";
    string lastName = "Doe";
    int age = 30;

    return (firstName, lastName, age);
}
```

The method `GetPerson` has been designed to return a `tuple` consisting of three values, the first and second values being of type `string`, and the third value being of type `int`. Once the `tuple` is returned, you may wonder how to access its individual elements.

We can use the default names (Item1, Item2...) for the `tuple` elements:
```cs
var person = GetPerson(Guid.NewGuid());

var firstName = person.Item1;
var lastName = person.Item2;
var age = person.Item3;

Console.WriteLine($"{firstName} {lastName} is {age} years old");
```

We can change the signature of the method to use custom names for the `tuple` elements:
```cs
(string FirstName, string LastName, int Age) GetPerson(Guid id)
{
    string firstName = "John";
    string lastName = "Doe";
    int age = 30;

    return (firstName, lastName, age);
}

var person = GetPerson(Guid.NewGuid());

var firstName = person.FirstName;
var lastName = person.LastName;
var age = person.Age;

Console.WriteLine($"{firstName} {lastName} is {age} years old");
```

However my personal preference for utilizing tuples, is to deconstruct them. Deconstruction is a feature that allows us to break apart a complex data structure, such as a `tuple`, into its individual parts and assign them to separate variables in a single statement. This makes code more concise and easier to read.

So, going back to our example, without changing anything in our method, we could deconstruct the `tuple` into seperate variables like this:

```cs
(string firstName, string lastName, int age) = GetPerson(Guid.NewGuid());
```

We could even use var, either inside or outside:
```cs
(var firstName, var lastName, var age) = GetPerson(Guid.NewGuid());
```

```cs
var (firstName, lastName, age) = GetPerson(Guid.NewGuid());
```

## Non-Tuple deconstruction

One of the major advantages of deconstruction is that it can be applied to any type, not just tuples! To implement a deconstructor, we simply need to create a method called Deconstruct with the following signature:
```cs
public void Deconstruct(out T1 x1, ..., out Tn xn) { ... }
```

Let's see in our example class.
```cs
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
    public DateTime Created { get; set; }
    public string Address { get; set; }

    public void Deconstruct(out string firstName, out string lastName, out int age)
    {
        firstName = FirstName;
        lastName = LastName;
        age = Age;
    }
}
```

At this point, we have everything we need to proceed. What's left to do is use a deconstruction assignment to extract the values of the `FirstName`, `LastName`, and `Age` properties of the `person` object and assign them to variables `firstName`, `lastName`, and `age`, respectively. This will allow us to access these values more conveniently and use them as needed.
```cs
var person = new Person
{
    FirstName = "John",
    LastName = "Doe",
    Age = 30,
    Address = "Random Address",
    DateOfBirth = DateTime.Now,
};

var (firstName, lastName, age) = person;
```

It is worth noting that we are not limited to just one `Deconstruct` method. In fact, we can define multiple `Deconstruct` methods for a given type. This can provide us with greater flexibility when it comes to deconstructing objects and accessing their individual parts. However, it is important to ensure that each `Deconstruct` method accepts a different number of parameters, since methods with the same number of parameters cannot be distinguished during overload resolution.

```cs
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
    public DateTime DateOfBirth { get; set; }
    public string Address { get; set; }

    public void Deconstruct(out string firstName, out string lastName, out int age)
    {
        firstName = FirstName;
        lastName = LastName;
        age = Age;
    }
    public void Deconstruct(out string firstName, out string lastName, out int age, out string address)
    {
        firstName = FirstName;
        lastName = LastName;
        age = Age;
        address = Address;
    }
}

var person = new Person
{
    FirstName = "John",
    LastName = "Doe",
    Age = 30,
    Address = "Random Address",
    DateOfBirth = DateTime.Now,
};
string firstName, lastName, address;
int age;

(firstName, lastName, age) = person;
Console.WriteLine($"{firstName} {lastName} is {age} years old");
// John Doe is 30 years old

(firstName, lastName, age, address) = person;
Console.WriteLine($"{firstName} {lastName} is {age} years old and is located at {address}");
// John Doe is 30 years old and is located at Random Address
```

And since i mentioned that the `Deconstruct` methods need to have different number of parameters, let's see what happens when they don't!
```cs
    public void Deconstruct(out string firstName, out string lastName, out int age)
    {
        firstName = FirstName;
        lastName = LastName;
        age = Age;
    }
    public void Deconstruct(out string firstName, out string lastName, out string address)
    {
        firstName = FirstName;
        lastName = LastName;
        address = Address;
    }
```

The difference here is in the last parameter, from `int` to `string`. This code does not compile and we get the following error:
```cs
The call is ambiguous between the following methods or properties: 'Person.Deconstruct(out string, out string, out int)' and 'Person.Deconstruct(out string, out string, out string)'
```

## Conclusion

Simply put, deconstruction is a highly effective tool in C# that can enhance your code's brevity and clarity. With this feature, you can swiftly extract values from objects and tuples and assign them to individual variables, saving you from writing repetitive code and making your code more efficient and comprehensible. If you haven't tried using deconstruction in your C# programming yet, it's definitely worth giving it a shot.