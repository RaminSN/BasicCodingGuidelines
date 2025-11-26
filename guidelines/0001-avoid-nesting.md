# Avoid Nesting

Nesting makes the code more difficult to read and is almost never necessary.

The simplest example and the easiest one to avoid is trivially nested if-statements.

```csharp
// Bad
if (foo)
{
    if (bar)
    {
        if (baz) 
        {
            quux();
        }
    }
}
```

```csharp
// Good
if (foo && bar && baz)
{
    quux();
}
```

Oftentimes the solution is not as easy as merging nested if-statements and requires more thinking. 
That doesn't make it any less important; writing clear code is an investment.

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
