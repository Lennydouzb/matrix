# Matrix

A simple generic matrix implementation in C++ using templates.

This project provides a `Matrix<T>` class supporting matrix creation, arithmetic operations, matrix multiplication, transposition, determinant calculation, vector dot/cross products, and common transformation matrices.

The implementation is entirely based on the C++ standard library and does not require external dependencies.

## Features

* Generic `Matrix<T>` implementation
* Identity matrices
* Custom matrix initialization
* Empty (zero-filled) matrices
* Translation matrices
* Scaling matrices
* Rotation matrices
* Matrix addition and subtraction
* Scalar addition, subtraction and multiplication
* Matrix multiplication
* Matrix transposition
* Determinant calculation
* Matrix minors
* Dot product
* Cross product
* Matrix type conversion
* Basic exception handling
* Matrix printing

---

## Requirements

* C++11 or later
* Standard C++ library

The implementation uses:

```cpp
#include <iostream>
#include <vector>
#include <initializer_list>
#include <cmath>
#include <string>
```

---

# Matrix Type

The matrix is implemented as a template:

```cpp
template <typename T>
class Matrix
```

This allows the matrix to store different numeric types, for example:

```cpp
Matrix<int>
Matrix<float>
Matrix<double>
```

Internally, matrix values are stored in a `std::vector<T>` in row-major order.

For a matrix:

```text
| 1 2 3 |
| 4 5 6 |
```

the internal data is:

```text
{ 1, 2, 3, 4, 5, 6 }
```

The element at row `i` and column `j` is stored at:

```cpp
data[i * cols + j]
```

---

# Matrix Construction

The class provides several ways to create a matrix.

## Identity Matrix

The constructor:

```cpp
Matrix(int cols)
```

creates a square identity matrix.

Example:

```cpp
Matrix<float> matrix(3);
```

Result:

```text
| 1 0 0 |
| 0 1 0 |
| 0 0 1 |
```

A value smaller than or equal to `0` throws `Matrix::TheException`.

---

## General Constructor

The main constructor is:

```cpp
Matrix(
    int rows,
    int cols,
    int type,
    std::initializer_list<T> list,
    int firstAxis = 0,
    int secondAxis = 0
);
```

The `type` parameter determines how the matrix is initialized.

The available types are defined by:

```cpp
enum e_types
{
    NO_TYPE,
    SCALE,
    TRANSLATION,
    ROTATION,
    EMPTY
};
```

The `firstAxis` and `secondAxis` parameters are primarily used by `ROTATION`.

---

# `NO_TYPE`

`NO_TYPE` creates a matrix directly from the values provided in the initializer list.

Example:

```cpp
Matrix<float> matrix(
    2,
    2,
    NO_TYPE,
    {
        1, 2,
        3, 4
    },
    0,0
);
```

Result:

```text
| 1 2 |
| 3 4 |
```

The initializer list must contain at least `rows * cols` values.

---

# `EMPTY`

`EMPTY` creates a matrix filled entirely with zeroes.

Example:

```cpp
Matrix<float> matrix(
    2,
    3,
    EMPTY,
    {},
    0,0
);
```

Result:

```text
| 0 0 0 |
| 0 0 0 |
```

This type is also used internally when creating a matrix that will subsequently be filled with calculated values.

---

# `TRANSLATION`

`TRANSLATION` creates a homogeneous translation matrix.

The matrix must be square and the initializer list must contain at least `rows - 1` values.

For a 4×4 matrix:

```cpp
Matrix<float> translation(
    4,
    4,
    TRANSLATION,
    {2, 3, 5},
    -1,
    -1
);
```

Result:

```text
| 1 0 0 2 |
| 0 1 0 3 |
| 0 0 1 5 |
| 0 0 0 1 |
```

The last two arguments are not used to select the translation direction; they can simply be left at their default values.

For a 3×3 matrix, the initializer list contains two translation values:

```cpp
Matrix<float> translation(
    3,
    3,
    TRANSLATION,
    {2, 3},
    0,0
);
```

Result:

```text
| 1 0 2 |
| 0 1 3 |
| 0 0 1 |
```

The implementation requires translation matrices to be square.

---

# `SCALE`

`SCALE` creates a homogeneous scaling matrix.

The matrix must be square and requires at least `rows - 1` scale values.

For a 3D transformation:

```cpp
Matrix<float> scale(
    4,
    4,
    SCALE,
    {2, 3, 4},
    -1,
    -1
);
```

Result:

```text
| 2 0 0 0 |
| 0 3 0 0 |
| 0 0 4 0 |
| 0 0 0 1 |
```

The final row/column represents the homogeneous coordinate and remains `1`.

---

# `ROTATION`

`ROTATION` creates a rotation matrix using two axes and an angle.

The constructor is:

```cpp
Matrix<float> rotation(
    4,
    4,
    ROTATION,
    {angle},
    firstAxis,
    secondAxis
);
```

The angle is provided directly to the constructor. The implementation calculates `cos(angle)` and `sin(angle)` internally.

The axes are represented by integer indices:

```text
X = 0
Y = 1
Z = 2
```

For example:

```cpp
Matrix<float> rotation(
    4,
    4,
    ROTATION,
    {angle},
    0,
    1
);
```

creates a rotation in the plane defined by the X and Y axes.

The two axes must be different and must be valid for the matrix dimensions.

The implementation checks:

* The matrix is square
* The matrix is at least 2×2
* `firstAxis != secondAxis`
* An angle is provided
* Both axes are inside the matrix dimensions

---

# Copying and Type Conversion

## Copy Constructor

A matrix can be copied directly:

```cpp
Matrix<float> original(
    2,
    2,
    NO_TYPE,
    {
        1, 2,
        3, 4
    },
    0,0
);

Matrix<float> copy(original);
```

---

## Converting Between Matrix Types

The class also provides a templated conversion constructor:

```cpp
template <typename U>
explicit Matrix(const Matrix<U>& other);
```

This allows values to be converted using `static_cast`.

Example:

```cpp
Matrix<int> integerMatrix(
    2,
    2,
    NO_TYPE,
    {
        1, 2,
        3, 4
    },
    0,0
);

Matrix<float> floatMatrix(integerMatrix);
```

---

# Arithmetic Operations

## Matrix Addition

Two matrices can be added with:

```cpp
matrix.add(other);
```

Both matrices must have identical dimensions.

Example:

```cpp
Matrix<int> a(
    2,
    2,
    NO_TYPE,
    {
        1, 2,
        3, 4
    },
    0,0
);

Matrix<int> b(
    2,
    2,
    NO_TYPE,
    {
        5, 6,
        7, 8
    },
    0,0
);

a.add(b);
```

`a` becomes:

```text
| 6  8  |
| 10 12 |
```

The operation modifies the current matrix.

---

## Scalar Addition

A scalar can be added to every element:

```cpp
matrix.add(5);
```

For:

```text
| 1 2 |
| 3 4 |
```

the result is:

```text
| 6 7 |
| 8 9 |
```

---

## Matrix Subtraction

Two matrices of the same dimensions can be subtracted:

```cpp
matrix.sub(other);
```

The operation modifies the current matrix.

---

## Scalar Subtraction

A scalar can also be subtracted from every element:

```cpp
matrix.sub(5);
```

---

# Matrix Multiplication

Matrix multiplication is performed using:

```cpp
matrix.multiply(other);
```

The operation follows the standard matrix multiplication rule.

For:

```text
A = m × n
B = n × p
```

the result has dimensions:

```text
m × p
```

Therefore:

```text
A.cols == B.rows
```

must be true.

Example:

```cpp
Matrix<int> a(
    2,
    3,
    NO_TYPE,
    {
        1, 2, 3,
        4, 5, 6
    },
    0,0
);

Matrix<int> b(
    3,
    2,
    NO_TYPE,
    {
        7, 8,
        9, 10,
        11, 12
    },
    0,0
);

a.multiply(b);
```

Result:

```text
| 58  64 |
| 139 154 |
```

The current matrix is replaced by the multiplication result, including its new dimensions.

---

## Scalar Multiplication

A matrix can be multiplied by a scalar:

```cpp
matrix.multiply(2);
```

Every element is multiplied by that value.

---

# Transpose

The `transpose()` method returns a new matrix with rows and columns exchanged.

```cpp
Matrix<int> matrix(
    2,
    3,
    NO_TYPE,
    {
        1, 2, 3,
        4, 5, 6
    },
    0,0
);

Matrix<int> transposed = matrix.transpose();
```

Result:

```text
| 1 4 |
| 2 5 |
| 3 6 |
```

The original matrix is not modified.

---

# Determinant

The determinant is calculated with:

```cpp
T det() const;
```

The matrix must be square.

The implementation contains direct calculations for:

* `1 × 1`
* `2 × 2`
* `3 × 3`

For larger matrices, the determinant is calculated recursively using matrix minors and cofactor expansion.

### 2×2 example

```cpp
Matrix<float> matrix(
    2,
    2,
    NO_TYPE,
    {
        1, 2,
        3, 4
    },
    0,0
);

float result = matrix.det();
```

The result is:

```text
-2
```

---

# Matrix Minor

The following method is available:

```cpp
Matrix<T> getMinor(int dropRow, int dropCol) const;
```

It returns a matrix with one row and one column removed.

For example, given:

```text
| 1 2 3 |
| 4 5 6 |
| 7 8 9 |
```

calling:

```cpp
matrix.getMinor(0, 1);
```

removes row `0` and column `1`, producing:

```text
| 4 6 |
| 7 9 |
```

This functionality is also used by the determinant implementation.

---

# Dot Product

The dot product is available through:

```cpp
T dot(const Matrix<T>& other) const;
```

Both matrices must be column vectors with the same number of rows.

Example:

```cpp
Matrix<float> a(
    3,
    1,
    NO_TYPE,
    {
        1,
        2,
        3
    },
    0,0
);

Matrix<float> b(
    3,
    1,
    NO_TYPE,
    {
        4,
        5,
        6
    },
    0,0
);

float result = a.dot(b);
```

Result:

```text
32
```

because:

```text
1 × 4 + 2 × 5 + 3 × 6 = 32
```

---

# Cross Product

The cross product is available through:

```cpp
Matrix<T> cross(const Matrix<T>& other) const;
```

It is restricted to `3 × 1` matrices representing 3D vectors.

Example:

```cpp
Matrix<float> a(
    3,
    1,
    NO_TYPE,
    {
        1,
        0,
        0
    },
    0,0
);

Matrix<float> b(
    3,
    1,
    NO_TYPE,
    {
        0,
        1,
        0
    },
    0,0
);

Matrix<float> result = a.cross(b);
```

Result:

```text
| 0 |
| 0 |
| 1 |
```

The implementation uses the standard 3D cross product:

```text
X = y1z2 - z1y2
Y = z1x2 - x1z2
Z = x1y2 - y1x2
```

---

# Accessors

The class provides several methods for accessing matrix information.

### Number of rows

```cpp
matrix.getRows();
```

Returns the number of rows.

### Number of columns

```cpp
matrix.getCols();
```

Returns the number of columns.

### Number of elements

```cpp
matrix.size();
```

Returns the number of stored elements.

### Matrix data

```cpp
matrix.getData();
```

Returns:

```cpp
const std::vector<T>&
```

giving read-only access to the internal data.

---

# Printing

The matrix can be printed using:

```cpp
matrix.print();
```

For example:

```cpp
Matrix<int> matrix(
    2,
    2,
    NO_TYPE,
    {
        1, 2,
        3, 4
    },
    0,0
);

matrix.print();
```

Output:

```text
| 1 2 |
| 3 4 |
```

---

# Exception Handling

The class defines its own exception type:

```cpp
class TheException : public std::exception
```

It stores an error message that can be retrieved with:

```cpp
e.what();
```

Example:

```cpp
try
{
    Matrix<float> matrix(0);
}
catch (const Matrix<float>::TheException& e)
{
    std::cerr << e.what() << std::endl;
}
```

The implementation throws exceptions for invalid operations such as:

* Invalid matrix dimensions
* Insufficient initialization values
* Non-square translation matrices
* Non-square scaling matrices
* Non-square rotation matrices
* Invalid rotation axes
* Rotation matrices smaller than 2×2
* Multiplication of incompatible matrices
* Determinant of a non-square matrix
* Dot product of incompatible vectors
* Cross product of matrices that are not `3 × 1`

---

# Example

A complete example using several features:

```cpp
#include "Matrix.hpp"

int main()
{
    Matrix<float> matrix(
        2,
        2,
        NO_TYPE,
        {
            1, 2,
            3, 4
        },
    0,0
    );

    matrix.print();

    Matrix<float> transposed = matrix.transpose();
    transposed.print();

    std::cout << "Determinant: "
              << matrix.det()
              << std::endl;

    return 0;
}
```

---

# Use Cases

This implementation can be used for projects involving:

* Linear algebra
* Vector mathematics
* Geometry
* Computer graphics
* 2D transformations
* 3D transformations
* OpenGL-related mathematics
* Homogeneous coordinates

The transformation constructors in particular make the class suitable for building transformation matrices for graphics applications.

---

# Project Structure

The core implementation is contained in:

```text
Matrix.hpp
```

The class is header-only, so no separate library compilation is required.

Include it directly in your project:

```cpp
#include "Matrix.hpp"
```
