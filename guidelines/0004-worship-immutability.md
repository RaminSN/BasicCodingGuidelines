# Worship Immutability

Moving from fragile and mutable to robust and immutable code is only a matter of slightly shifting your mental model.

Eric is a star employee who is about to get his long-awaited yearly raise of 3%.

```csharp
var eric = GetEmployee("660d40fc-3693-4134-bf54-ab738b5652fd");

// eric.Salary == 1298

ModifySalary(eric, 1.03m);

// eric.Salary == 1337
```

Here we think of `eric` as a living changing entity whose salary changes.

Now, consider this alternative approach:

```csharp
var eric = GetEmployee("660d40fc-3693-4134-bf54-ab738b5652fd");

// eric.Salary == 1298

var newEric = ModifySalary(eric, 1.03m);

// eric.Salary == 1298
// newEric.Salary == 1337
```

Here we treat our data structure as actual data. We have two snapshots of Eric from different points in time, and these snapshots are set in stone.
They are facts about Eric that are both true for different points in time. They are _immutable_.

The salary of `eric` never changes. A piece of code (such as the `ModifySalary` function) that receives the `eric` variable can be confident that the
salary of `eric` won't change unexpectedly during its calculations.

Let's drive home the point with a more complete example.

Here we have code that follows the bad mental model where objects are "living" and "changing".
Our code consists of impure function that mutate state.
```csharp
void ModifySalary(Employee employee, decimal factor)
{
    if (!VerifyWorkedDays(employee))
    {
        throw new InvalidOperationException("Ineligible for salary increase.");
    }

    employee.Salary *= factor;
}

bool VerifyWorkedDays(Employee employee)
{
    ExcludeUnpaidLeave(employee);
    return employee.WorkedDaysThisYear >= 220;
}

void ExcludeUnpaidLeave(Employee employee)
{
    // Mutates data
    employee.WorkedDaysThisYear -= employee.UnpaidLeaveDays;
}
```
We weren't careful enough and caused a bug. By modifying the salary of Eric we accidentally
also modified his work days.
```csharp
var eric = GetEmployee("660d40fc-3693-4134-bf54-ab738b5652fd");

// eric.Salary == 1298
// eric.WorkedDaysThisYear == 240

ModifySalary(eric, 1.03m);

// eric.Salary == 1337
// eric.WorkedDaysThisYear == 230
```
Notice that immutability and purity (the previous guideline) are two sides of the same golden coin.

If `eric` were an immutable object, our mutation wouldn't be possible and we would
be forced to write better code by creating `newEric` as before, resulting in purer code.

---

Have you ever experienced being deep inside a debugging rabbit hole chasing a variable through the code
in order to figure out how and why it ends up with an unexpected value? If you can relate to that pain,
immutability should resonate with you.