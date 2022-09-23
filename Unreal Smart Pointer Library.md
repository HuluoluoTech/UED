## Smart Pointer Types
* Shared Pointers (TSharedPtr)
A Shared Pointer owns the object it references, indefinitely preventing deletion of that object, and ultimately handling its deletion when no Shared Pointer or Shared Reference (see below) references it. A Shared Pointer can be empty, meaning it doesn't reference any object. Any non-null Shared Pointer can produce a Shared Reference to the object it references.

* Shared References (TSharedRef)
A Shared Reference acts like a Shared Pointer, in the sense that it owns the object it references. They differ with regard to null objects; Shared References must always reference a non-null object. Because Shared Pointers don't have that restriction, a Shared Reference can always be converted to a Shared Pointer, and that Shared Pointer is guaranteed to reference a valid object. Use Shared References when you want a guarantee that the referenced object is non-null, or if you want to indicate shared object ownership.

* Weak Pointers (TWeakPtr)
Weak Pointers are similar to Shared Pointers, but do not own the object they reference, and therefore do not affect its lifecycle. This property can be very useful, as it breaks reference cycles, but it also means that a Weak Pointer can become null at any time, without warning. For this reason, a Weak Pointer can produce a Shared Pointer to the object it references, ensuring programmers safe access to the object on a temporary basis.

* Unique Pointers (TUniquePtr)
A Unique Pointer solely and explicitly owns the object it references. Since there can only be one Unique Pointer to a given resource, Unique Pointers can transfer ownership, but cannot share it. Any attempts to copy a Unique Pointer will result in a compile error. When a Unique Pointer goes out of scope, it will automatically delete the object it references.

### Warning
**Making a Shared Pointer or Shared Reference to an object that a Unique Pointer references is dangerous. This will not suspend the Unique Pointer's behavior of deleting the object upon its own destruction, even though other Smart Pointers still reference it. Similarly, you should not make a Unique Pointer to an object that is referenced by a Shared Pointer or Shared Reference.**

## Benefits
* Prevents memory leaks
Smart Pointers (other than Weak Pointers) automatically delete objects when there are no more shared references.

* Weak referencing
Weak Pointers break reference cycles and prevent dangling pointers.

* Optional Thread safety
The Unreal Smart Pointer Library includes thread-safe code that manages reference counting across multiple threads. Thread safety can be traded out for better performance if it isn't needed.

* Runtime safety
Shared References are never null and can always be dereferenced.

* Confers intent
You can easily tell an object owner from an observer.

* Memory
Smart Pointers are only twice the size of a C++ pointer in 64-bit (plus a shared 16-byte reference controller). The exception to this is **Unique Pointers, which are the same size as C++ pointers.**

## Smart Pointer Implementation Details
Smart Pointers in the Unreal Smart Pointer Library all share some general characteristics in terms of functionality and efficiency.

### Speed
Always keep performance in mind when considering using Smart Pointers. Smart Pointers are well-suited for certain high-level systems, resource management, or tools programming. However, some Smart Pointer types are slower than raw C++ pointers, and this overhead makes them less useful in low-level engine code, such as rendering.

Some of the general **performance benefits** of Smart Pointers are:

* All operations are constant time.
* Dereferencing most Smart Pointers is just as fast as raw C++ pointers (in shipping builds).
* Copying Smart Pointers never allocates memory.
* Thread-safe Smart Pointers are lockless.

Performance **drawbacks of Smart Pointers** include:

* Creating and copying Smart Pointers involves more overhead than creating and copying raw C++ pointers.
* Maintaining reference counts adds cycles to basic operations.
* Some Smart Pointers use more memory than raw C++ pointers.
* There are two heap allocations for reference controllers. **Using MakeShared instead of MakeShareable avoids the second allocation**, and can improve performance.

### Intrusive Accessors
Shared pointers are non-intrusive, which means the object does not know whether or not a Smart Pointer owns it. This is usually acceptable, but there may be cases in which you want to access the object as a Shared Reference or Shared Pointer. To do this, derive the object's class from TSharedFromThis, using the object's class as the template parameter. TSharedFromThis provides two functions, AsShared and SharedThis, that can convert the object to a Shared Reference (and from there, to a Shared Pointer). This can be useful with class factories that always return Shared References, or when you need to pass your object to a system that expects a Shared Reference or Shared Pointer. AsShared will return your class as the type originally passed as the template argument to TSharedFromThis, which may be a parent type to the calling object, while SharedThis will derive the type directly from this and return a Smart Pointer referencing an object of that type.

### Casting
* Up-casting is implicit
* const cast with the ConstCastSharedPtr function
* static cast (often to downcast to derived class pointers) with StaticCastSharedPtr
* **Dynamic casting is not supported**

### ThreadSafe
* TSharedPtr<T, ESPMode::ThreadSafe>
* TSharedRef<T, ESPMode::ThreadSafe>
* TWeakPtr<T, ESPMode::ThreadSafe>
* TSharedFromThis<T, ESPMode::ThreadSafe>

**thread-safe versions are a bit slower than the defaults due to atomic reference counting**

## Tips
* Avoid passing data to functions as TSharedRef or TSharedPtr parameters where possible
* You can forward-declare Shared Pointers to incomplete types.
* **Shared Pointers are not compatible with Unreal objects (UObject and its derived classes). **

### Reference
[Smart Pointer In Unreal Engine](https://docs.unrealengine.com/5.0/en-US/smart-pointers-in-unreal-engine/)
