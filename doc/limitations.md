# Limitations

## Nameof

* If argument does not have name, occurs the compilation error `"Expression does not have a name."`.

## Nameof Type

* To check is nameof type supported compiler use macro `NAMEOF_TYPE_SUPPORTED` or constexpr constant `nameof::is_nameof_type_supported`.

* This library uses a compiler-specific hack (based on `__PRETTY_FUNCTION__` / `__FUNCSIG__`), which works on Clang >= 5, MSVC >= 15.3 and GCC >= 7.

* Nameof type returns compiler-specific type name.

* If argument does not have name, occurs the compilation error `"Expression does not have a name."`.

## Nameof Enum

* To check is nameof enum supported compiler use macro `NAMEOF_ENUM_SUPPORTED` or constexpr constant `nameof::is_nameof_enum_supported`.

* This library uses a compiler-specific hack (based on `__PRETTY_FUNCTION__` / `__FUNCSIG__`), which works on Clang >= 5, MSVC >= 15.3 and GCC >= 9.

* Enum can't reflect if the enum is a forward declaration.

* Enum value must be in range `[NAMEOF_ENUM_RANGE_MIN, NAMEOF_ENUM_RANGE_MAX]`.

  * By default `NAMEOF_ENUM_RANGE_MIN = -128`, `NAMEOF_ENUM_RANGE_MAX = 128`.

  * `NAMEOF_ENUM_RANGE_MIN` must be less or equals than `0` and must be greater than `INT16_MIN`.

  * `NAMEOF_ENUM_RANGE_MAX` must be greater than `0` and must be less than `INT16_MAX`.

  * If need another range for all enum types by default, redefine the macro `NAMEOF_ENUM_RANGE_MIN` and `NAMEOF_ENUM_RANGE_MAX`.

    ```cpp
    #define NAMEOF_ENUM_RANGE_MIN 0
    #define NAMEOF_ENUM_RANGE_MAX 256
    #include <nameof.hpp>
    ```

  * If need another range for specific enum type, add specialization `enum_range` for necessary enum type. Specialization of `enum_range` must be injected in `namespace nameof`.

    ```cpp
    #include <nameof.hpp>

    enum class number { one = 100, two = 200, three = 300 };

    namespace nameof {
    template <>
    struct enum_range<number> {
      static constexpr int min = 100;
      static constexpr int max = 300;
    };
    } // namespace nameof
    ```

* If you hit a message like this:

  ```text
  [...]
  note: constexpr evaluation hit maximum step limit; possible infinite loop?
  ```

  Change the limit for the number of constexpr evaluated:
  * MSVC `/constexpr:depthN`, `/constexpr:stepsN` <https://docs.microsoft.com/en-us/cpp/build/reference/constexpr-control-constexpr-evaluation>
  * Clang `-fconstexpr-depth=N`, `-fconstexpr-steps=N` <https://clang.llvm.org/docs/UsersManual.html#controlling-implementation-limits>
  * GCC `-fconstexpr-depth=N`, `-fconstexpr-loop-limit=N`, `-fconstexpr-ops-limit=N` <https://gcc.gnu.org/onlinedocs/gcc-9.2.0/gcc/C_002b_002b-Dialect-Options.html>

* Nameof enum obtains the first defined value enums, and won't work if value are aliased.

  ```cpp
  enum ShapeKind {
    ConvexBegin = 0,
    Box = 0, // Won't work.
    Sphere = 1,
    ConvexEnd = 2,
    Donut = 2, // Won't work too.
    Banana = 3,
    COUNT = 4,
  };
  // nameof::nameof_enum(ShapeKind::Box) -> "ConvexBegin"
  // NAMEOF_ENUM(ShapeKind::Box) -> "ConvexBegin"
  ```

  Work around the issue:

  ```cpp
  enum ShapeKind {
    // Convex shapes, see ConvexBegin and ConvexEnd below.
    Box = 0,
    Sphere = 1,

    // Non-convex shapes.
    Donut = 2,
    Banana = 3,

    COUNT = Banana + 1,

    // Non-reflected aliases.
    ConvexBegin = Box,
    ConvexEnd = Sphere + 1,
  };
  // nameof::nameof_enum(ShapeKind::Box) -> "Box"
  // NAMEOF_ENUM(ShapeKind::Box) -> "Box"

  // Non-reflected aliases.
  // nameof::nameof_enum(ShapeKind::ConvexBegin) -> "Box"
  // NAMEOF_ENUM(ShapeKind::ConvexBegin) -> "Box"
  ```
