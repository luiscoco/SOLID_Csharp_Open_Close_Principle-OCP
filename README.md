# SOLID_Csharp_Open_Close_Principle-OCP

## Open/Closed Principle (OCP):
The Open/Closed Principle suggests that software entities (such as classes, modules, functions) should be open for extension but closed for modification. 
This means you should be able to add new functionality to a system without altering existing code, promoting reusability and reducing the risk of introducing bugs.

```csharp
﻿using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.CompilerServices;
using static System.Console;

namespace DotNetDesignPatterns.SOLID.OCP;

public enum Color  { Red, Green, Blue }

public enum Size { Small, Medium, Large, Yuge }

public record Product(string Name, Color Color, Size Size);

public class Demo
{
  static void Main(string[] args)
  {
    var apple = new Product("Apple", Color.Green, Size.Small);
    var tree = new Product("Tree", Color.Green, Size.Large);
    var house = new Product("House", Color.Blue, Size.Large);

    Product[] products = {apple, tree, house};

    var pf = new ProductFilter();
    
    WriteLine("Green products (old):");
    foreach (var p in pf.FilterByColor(products, Color.Green))
      WriteLine($" - {p.Name} is green");

    // ^^ BEFORE

    // vv AFTER
    var bf = new BetterFilter();
    WriteLine("Green products (new):");
    foreach (var p in bf.Filter(products, new ColorSpecification(Color.Green)))
      WriteLine($" - {p.Name} is green");

    WriteLine("Large products");
    foreach (var p in bf.Filter(products, new SizeSpecification(Size.Large)))
      WriteLine($" - {p.Name} is large");

    var largeGreenSpec = new ColorSpecification(Color.Green) 
                         & new SizeSpecification(Size.Large);
    //var largeGreenSpec = Color.Green.And(Size.Large);
    
    WriteLine("Large blue items");
    foreach (var p in bf.Filter(products,
      new AndSpecification<Product>(new ColorSpecification(Color.Blue), 
        new SizeSpecification(Size.Large)))
    )
    {
      WriteLine($" - {p.Name} is big and blue");
    }
  }
}

public class ProductFilter
{
  // let's suppose we don't want ad-hoc queries on products
  public IEnumerable<Product> FilterByColor(IEnumerable<Product> products, Color color)
  {
    foreach (var p in products)
      if (p.Color == color)
        yield return p;
  }
  
  public static IEnumerable<Product> FilterBySize(IEnumerable<Product> products, Size size)
  {
    foreach (var p in products)
      if (p.Size == size)
        yield return p;
  }

  public static IEnumerable<Product> FilterBySizeAndColor(IEnumerable<Product> products, Size size, Color color)
  {
    foreach (var p in products)
      if (p.Size == size && p.Color == color)
        yield return p;
  } // state space explosion
    // 3 criteria = 7 methods

  // OCP = open for extension but closed for modification
}

// we introduce two new interfaces that are open for extension

public interface IFilter<T>
{
  IEnumerable<T> Filter(IEnumerable<T> items, Specification<T> spec);
}

public class ColorSpecification : Specification<Product>
{
  private readonly Color color;

  public ColorSpecification(Color color)
  {
    this.color = color;
  }

  public override bool IsSatisfied(Product p)
  {
    return p.Color == color;
  }
}

public class SizeSpecification : Specification<Product>
{
  private readonly Size size;

  public SizeSpecification(Size size)
  {
    this.size = size;
  }

  public override bool IsSatisfied(Product p)
  {
    return p.Size == size;
  }
}

public abstract class Specification<T>
{
  public abstract bool IsSatisfied(T p);

  public static Specification<T> operator &(
    Specification<T> first, Specification<T> second)
  {
    return new AndSpecification<T>(first, second);
  }
}

public abstract class CompositeSpecification<T> : Specification<T>
{
  protected readonly Specification<T>[] items;

  public CompositeSpecification(params Specification<T>[] items)
  {
    this.items = items;
  }
}

// combinator
public class AndSpecification<T> : CompositeSpecification<T>
{
  public AndSpecification(params Specification<T>[] items) : base(items)
  {
  }

  public override bool IsSatisfied(T t)
  {
    // Any -> OrSpecification
    return items.All(i => i.IsSatisfied(t));
  }
}

public static class All
{
  [MethodImpl(MethodImplOptions.AggressiveInlining)]
  public static AndSpecification<T> Of<T>(params Specification<T>[] items)
  {
    return new AndSpecification<T>(items);
  }
}

public static class CompositeSpecificationExtensions
{
  // extension method on typed value tuple
  public static AndSpecification<T> All<T>(
    this (Specification<T> first, Specification<T> second) specs)
  {
    return new AndSpecification<T>(specs.first, specs.second);
  }
}

public class BetterFilter : IFilter<Product>
{
  public IEnumerable<Product> Filter(
    IEnumerable<Product> items, Specification<Product> spec)
  {
    foreach (var i in items)
      if (spec.IsSatisfied(i))
        yield return i;
  }
}

public static class CriteriaExtensions
{
  public static AndSpecification<Product> And(this Color color, Size size)
  {
    return new AndSpecification<Product>(
      new ColorSpecification(color),
      new SizeSpecification(size));
  }
}
```
