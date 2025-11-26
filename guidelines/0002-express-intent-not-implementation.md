# Express Intent, Not Implementation

This is another way of saying _"Favor Declarative Over Imperative"_.

The following two instructions have the same result. Which one is more clear?

1. >Give me the name of the oldest student in the mathematics class.

2. >Look at each student in the mathematics class while keeping track of oldest one you have seen so far. When you are done, give me the name of the tracked student.

The first one is expressing intent. It expresses _what_ the author wanted. There is little room for confusion
on the reader's part regarding what the author sought to communicate.

The second one is expressing implementation. It expreses _how_ to achieve what the author wanted. Not only is it more complicated for
the author to conjure up such an instruction, it is also much more difficult to understand.

Code-equivalents of the two instructions above could look like this:

```csharp
string result = mathClass
    .Students
    .OrderByDescending(student => student.Age)
    .First()
    .Name;
   ```

```csharp
Student oldestStudent = mathClass.First();
foreach (var student in mathClass.Students)
{
    if (student.Age > oldestStudent.Age)
        oldestStudent = student;
}

string result = oldestStudent.Name;
```
---
Going back to the example from our discussion regarding nesting, we see a prime example of this.
```csharp
// Bad
List<Person> persons = new List<Person>();
foreach (var country in countries)
{
    if (country.Population >= 1_000_000)
    {
        foreach (city in country.Cities)
        {
            if (city.Population < 20_000)
            {
                foreach (person in city.Inhabitants)
                {
                    if (person.Age >= 20)
                    {
                        persons.Add(person);    
                    }        
                }
            }
        }
    }
}
```

```csharp
// Good
List<Person> persons = countries
    .Where(country => country.Population >= 1_000_000)
    .SelectMany(country => country.Cities)
    .Where(city => city.Population < 20_000)
    .SelectMany(city => city.Inhabitants)
    .Where(person => person.Age >= 20)
    .ToList();
```

To make this even more declarative, we could name things:

```csharp
// Even better
bool IsPandemicRiskZone(Country country) => country.Population >= 1_000_000;
bool HasOnlySmallHospitals(City city) => city.Population < 20_000;
bool IsAdult(Person person) => person.Age >= 20;

List<Person> priorityTargets = countries
    .Where(IsPandemicRiskZone)
    .SelectMany(country => country.Cities)
    .Where(HasOnlySmallHospitals)
    .SelectMany(city => city.Inhabitants)
    .Where(IsAdult)
    .ToList();
```

It is now much clearer what the _intent_ of the author of this code was. (He is attempting to
predict Scarecrow's next targets to give Batman a head start!) We also feel a lot safer knowing that there
is no esoteric edge-case that we are missing. There is simply no room for surprises in simple declarative code.