
# Architecture and Optimization
## Code Optimization Practice

### Objective:
Enhance the performance of provided code snippets by applying code-level optimization techniques manually. This lab focuses on deepening your understanding and application of optimization principles in coding.

### Introduction:
At the beginning of the lab, you will receive a set of code snippets that contain common performance issues. These snippets will come from different programming languages, including JavaScript, Java, and C#, to cover a variety of scenarios and common inefficiencies found in real-world programming.

### JavaScript Snippet:

```javascript
// Inefficient loop handling and excessive DOM manipulation
function updateList(items) {
  let list = document.getElementById("itemList");
  list.innerHTML = "";
  for (let i = 0; i < items.length; i++) {
    let listItem = document.createElement("li");
    listItem.innerHTML = items[i];
    list.appendChild(listItem);
  }
}
```
---
### Optimized code:

```javascript
function updateList(items) {
  const list = document.getElementById("itemList");

  // Create a fragment to avoid multiple DOM manipulations
  const fragment = document.createDocumentFragment();

  // Generate new list items
  items.forEach(item => {
    const listItem = document.createElement("li");
    listItem.textContent = item; // More secure than innerHTML
    fragment.appendChild(listItem);
  });

  // Clear the list and add new items in one go
  list.innerHTML = ""; // Efficiently cleans existing content
  list.appendChild(fragment); // Update the DOM in a single operation
}
```
- createDocumentFragment is used to avoid excessive manipulation of the DOM, which improves performance.
- Instead of making use of a `for` loop, the `forEach` method is used to iterate over the elements of the `items` array. This is more readable and less error prone.
- `TextContent` is used instead of `innerHTML` to avoid HTML rendering and improve security by preventing malicious code injection.

### Java Snippet:

```java 
// Redundant database queries
public class ProductLoader {
    public List<Product> loadProducts() {
        List<Product> products = new ArrayList<>();
        for (int id = 1; id <= 100; id++) {
            products.add(database.getProductById(id));
        }
        return products;
    }
}
```

- A serious problem with the original code is that 100 queries are made to the database, one for each product. This can be very inefficient and slow, especially if the database is on a remote network.
- It is fully coupled to the database implementation, which makes it difficult to test and evolve the code.
- It does not adequately handle errors that may occur during product loading.
- It does not validate if the list of IDs is empty or null.
- Dependency injection is not used to provide flexibility and facilitate testing.
- Failure to use a functional programming approach to simplify the code and make it more readable.
---
### Optimized code:

```java
public class ProductLoader {

    private final ProductRepository productRepository;

    // Dependency injection for flexibility and stability
    public ProductLoader(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public List<Product> loadProducts(List<Integer> ids) {
        if (ids == null || ids.isEmpty()) { // Validating Arguments
            throw new IllegalArgumentException("The list of IDs cannot be empty"); // Throws exception
        }

        try {
            // Get products in a single optimized query using jpa
            return productRepository.findProductsByIds(ids);
        } catch (Exception e) {
            // Handling errors properly
            throw new RuntimeException("Error al cargar productos: " + e.getMessage(), e); // Throw custom exception
        }
    }

    public interface ProductRepository extends JpaRepository<Product, Integer> { // Interface for database abstraction

        List<Product> findProductsByIds(List<Integer> ids);// Method to search for products by Ids
    }
}
```
- Dependency injection is used to decouple the `ProductLoader` class from the concrete database implementation. This facilitates unit testing and allows changing the database implementation without modifying the `ProductLoader` code.
- The `loadProducts` method is changed to accept a list of IDs as an argument. This allows to load specific products instead of all products in a fixed range.
- Validates if the ID list is null or empty and throws an exception if so. This avoids unexpected errors and improves the robustness of the code.
- Errors are handled properly by using a `try-catch` block to catch exceptions and throw a custom exception in case of error.
- A `ProductRepository` interface is introduced to abstract the database and provide an extension point for different database implementations.
- A `findProductsByIds` method is added to the `ProductRepository` interface to search for products by a list of IDs. This allows an optimized query to load products in a single operation.

Alternatively, if I need a fixed range of ids:

```java
import java.util.List;

public List<Product> loadProducts(int startId, int endId) {
    if (startId > endId) {
        throw new IllegalArgumentException("The start ID cannot be greater than the end Id.");
    }

    List<Integer> rangeIds = IntStream.rangeClosed(startId, endId)
            .boxed() // Convert to Stream<Integer>
            .collect(Collectors.toList()); // Generate Ids List

    try {
        // Get products in a single optimized query using jpa
        return productRepository.findProductsByIds(rangeIds);
    } catch (Exception e) {
        // Handling errors properly
        throw new RuntimeException("Error loading products: " + e.getMessage(), e); // Throws custom exception
    }
}
```
- A new `loadProducts` method is added that accepts a range of IDs (start and end) as arguments. This allows loading products in a specific range of IDs instead of all products in a fixed range.
- It validates if the start ID is greater than the end ID and throws an exception if it is. This avoids logical errors and improves code robustness.
- A list of IDs in the specified range is generated using `IntStream.rangeClosed`, `boxed` and `collect(Collectors.toList)`. This simplifies the generation of the list of IDs and improves the readability of the code.
- The `findProductsByIds` method of the `ProductRepository` interface is used to search for products by a list of IDs in a specific range. This allows an optimized query to load the products in a single operation.

### C# Snippet:

```csharp
// Unnecessary computations in data processing
public List<int> ProcessData(List<int> data) {
    List<int> result = new List<int>();
    foreach (var d in data) {
        if (d % 2 == 0) {
            result.Add(d * 2);
        } else {
            result.Add(d * 3);
        }
    }
    return result;
}
```
- It does not adequately handle errors that may occur during data processing.
- The original code performs unnecessary calculations when processing the data. Multiplies even numbers by 2 and odd numbers by 3, which can be inefficient and redundant.

---
### Código optimizado:

```csharp
public List<int> ProcessData(List<int> data)
{
    return data.Select(d => d * (d % 2 == 0 ? 2 : 3)).ToList();
}
```
- The need for an `if-else` block is eliminated and the calculation is performed in a more concise and readable way using the ternary conditional operator `? :`.
- The `ToList` method is used to convert the result to a list of integers and return it as the result of the function.
- The use of a `foreach` loop is avoided and the `Select` method is used to apply the transformation to each element of the `data` list.
- LINQ (Language Integrated Query) is used to simplify data processing. The lambda expression `d => d * (d % 2 == 0 ? 2 : 3)` performs the necessary calculation in a single line of code.
