# Avoid Nesting

Nesting makes the code more difficult to read and is almost never necessary. While nesting in and of itself does not make code less readable,
it is a good general rule to follow for imperative code that easily ends up with many logical branches.

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

In the above example, one could argue that the nested form is not really unreadable. That is arguably true as the code really only has one branch and
is easy to follow despite the nesting.

Here is a more realistic example:

```csharp
decimal GetInvoiceAmount(bool includeVat, bool shouldRound, bool alwaysPositive, Invoice invoice)
{
    decimal result;
    if (alwaysPositive)
    {
        if (shouldRound)
        {
            if (includeVat)
            {
                result = Math.Abs(Math.Round(invoice.AmountIncludingVat));    
            }
            else
            {
                result = Math.Abs(Math.Round(invoice.AmountExcludingVat));
            }
        }
        else
        {
            if (includeVat)
            {
                result = Math.Abs(invoice.AmountIncludingVat);    
            }
            else
            {
                result = Math.Abs(invoice.AmountExcludingVat);
            }
        }
    }
    else
    {
        if (shouldRound)
        {
            if (includeVat)
            {
                result = Math.Round(invoice.AmountIncludingVat);    
            }
            else
            {
                result = Math.Round(invoice.AmountIncludingVat);
            }
        }
        else
        {
            if (includeVat)
            {
                result = invoice.AmountIncludingVat;    
            }
            else
            {
                result = invoice.AmountExcludingVat;
            }
        }
    }
    return result;
}
```
There is a bug in the above example. How easy was it to spot?

Here's the same logic but written with less nesting (and bug-free):

```csharp
decimal GetInvoiceAmount(bool includeVat, bool shouldRound, bool alwaysPositive, Invoice invoice)
{
    decimal result;

    if (includeVat)
    {
        result = invoice.AmountIncludingVat;
    }
    else
    {
        result = invoice.AmountExcludingVat;
    }

    if (shouldRound)
    {
        result = Math.Round(result);
    }

    if (alwaysPositive)
    {
        result = Math.Abs(result);
    }

    return result;
}
```