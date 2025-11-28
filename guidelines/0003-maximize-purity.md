# Maximize Purity

Pure functions are functions that have no side effects and always return the same thing for a given input.

```csharp
string BaseURL { get; set; } = "https://example.com";

// Impure! Is dependent on an external value and can therefore return different values for the same input.
bool IsValidUrl(string path)
{
    var url = $"{BaseURL}{path}";

    return url.Length < 30;
}
```

```csharp

// Impure! It returns different values depending on the result of the request. In addition, the request itself is a side-effect and therefore impure.
bool IsValidUrl(string baseUrl, string path)
{
    var url = $"{baseUrl}{path}";

    var isOK = SendRequestSync(url).Response.StatusCode == 200;
    
    return isOK && url.Length < 30;
}
```


```csharp

// Pure!
bool IsValidUrl(string baseUrl, string path)
{
    var url = $"{baseUrl}{path}";
    
    return url.Length < 30;
}
```


```csharp

// Pure!
bool IsValidUrl(string url, HttpResponse response)
{
    return response.StatusCode == 200 && url.Length < 30;
}
```


Pure functions have a set of valuable characteristics.

* **Effective testing:**  
    
    Since pure functions always return the same value for a given input, testing them is trivial and highly effective.

* **Safe:**

    Calling a pure function has no side effects and is therefore risk-free. This safety is valuable for testing as well as writing business logic.

* **Indefinite caching:**

    Since the output for a given input is always the same, the function calls can be cached without risk of the cache becoming invalid. Caching
    pure functions is a common technique known as _memoization_.

---
In order to maximize purity (and thus minimizing impurity), impure functions that also contain pure logic should be refactored so that the size of the impure function is minimized.

```csharp
// Bad

// Large impure function
void ProcessOrder(decimal price, int quantity, string customerEmail)
{
    var loyaltyPoints = Database.GetLoyaltyPoints(customerEmail);
    
    decimal subtotal = price * quantity;
    decimal discount = subtotal > 100 ? subtotal * 0.1m : 0;
    
    if (loyaltyPoints > 500)
        discount += 20;
    
    decimal finalPrice = subtotal - discount;
    int newPoints = (int)(finalPrice / 10);
    
    Database.AddLoyaltyPoints(customerEmail, newPoints);
    SendEmail(customerEmail, $"Total ${finalPrice}, earned {newPoints} points");
}
```

```csharp
// Good

// Pure!
OrderResult CalculateOrder(decimal price, int quantity, int currentPoints)
{
    decimal subtotal = price * quantity;
    decimal discount = subtotal > 100 ? subtotal * 0.1m : 0;
    
    if (currentPoints > 500)
        discount += 20;
    
    decimal finalPrice = subtotal - discount;
    int pointsEarned = (int)(finalPrice / 10);
    
    return new OrderResult 
    { 
        FinalPrice = finalPrice, 
        PointsEarned = pointsEarned 
    };
}

// Small impure function
void ProcessOrder(decimal price, int quantity, string customerEmail)
{
    var loyaltyPoints = Database.GetLoyaltyPoints(customerEmail);
    var result = CalculateOrder(price, quantity, loyaltyPoints);
    
    Database.AddLoyaltyPoints(customerEmail, result.PointsEarned);
    SendEmail(customerEmail, $"Total ${result.FinalPrice}, earned {result.PointsEarned} points");
}
```

Impurity is necessary for a program to do something useful. The key is to keep the core business logic pure and drive the impurities toward the edges of our applications.
