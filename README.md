## Overview

This project implements **Sparse Matrix Vector Multiplication** using the **Compressed Sparse Row (CSR)** format in C.

The main goal is to optimize matrix vector multiplication by storing only **non zero** elements instead of the **full matrix**. This significantly reduces computation time and memory usage when dealing with sparse matrices.

The implementation is validated using a randomized test harness that compares the CSR based result against a brute force dense matrix multiplication approach.

## CSR Representation

CSR (Compressed Sparse Row) format stores only non zero elements of a matrix to improve efficiency.

Instead of storing the full matrix, we use three arrays:

- values[] → stores all non-zero elements
- col_indices[] → stores column index of each value
- row_ptrs[] → stores starting index of each row in values[]

## Implementation

The core function `sparse_multiply()` performs two main tasks:

### 1. Conversion from Dense to CSR
- The input matrix is scanned row by row
- Only non zero elements are stored
- Their values and column indices are saved
- `row_ptrs` is updated to mark row boundaries

```
 int nnz = 0;

    // CSR structure
    row_ptrs[0] = 0;

    for (int i = 0; i < rows; i++) {
	for (int j = 0; j < cols; j++) {
	    double val = A[i * cols + j];

	    if (val != 0.0) {
		values[nnz] = val;
		col_indices[nnz] = j;
		nnz++;
	    }
	}
	row_ptrs[i + 1] = nnz;
    }

    *out_nnz = nnz;
```
## Step by step Explanation
 **1.1 Initialize non zero counter**

    int nnz = 0;

- nnz = number of non-zero elements
- This tracks how many values we store in CSR arrays
- Starts from 0 because we haven’t stored anything yet

**1.2 Initialize CSR row pointer**

    row_ptrs[0] = 0;

- row_ptrs stores where each row starts in CSR format
- First row always starts at index 0 in values[]

**1.3 Loop through matrix rows**

    for (int i = 0; i < rows; i++) {

- Go row by row in matrix A
- i = current row index

**1.4 Loop through columns**

    for (int j = 0; j < cols; j++) {

- Scan every element in row i
- j = column index

**1.5 Access matrix element**

    double val = A[i * cols + j];

- Matrix is stored as 1D array
- Convert 2D index → 1D index

Formula:

    row-major indexing = i * cols + j

- val = current element in matrix

**1.6 Check if element is non-zero**

    if (val != 0.0) {

- CSR ignores zero values
- Only store meaningful (non-zero) data

**1.7 Store value in CSR array**

    values[nnz] = val;

- Store actual matrix value in CSR values[]

**1.8 Store column index**

    col_indices[nnz] = j;

- Store which column this value came from
- Needed later during multiplication

**1.9 Increment nnz**

    nnz++;

- We stored one more non-zero element
- Move to next CSR position

**1.10 End of row update**

    row_ptrs[i + 1] = nnz;

- Marks end of current row in CSR structure

Tells:

    “Row i ends at index nnz in values[]”

- So next row starts from here

**1.11 Save total non zeros**

    *out_nnz = nnz;

- Return total number of non-zero elements to caller
- Stored via pointer (out_nnz)

### 2. Sparse Matrix Vector Multiplication
- For each row, only non zero elements are processed
- Each value is multiplied with the corresponding element of vector `x`
- Results are accumulated into output vector `y`
```
// compute y = A * x using CSR
    for (int i = 0; i < rows; i++) {
	double sum = 0.0;

	for (int k = row_ptrs[i]; k < row_ptrs[i + 1]; k++) {
	    sum += values[k] * x[col_indices[k]];
	}

	y[i] = sum;
    }
```
**2.1 Loop through each row**

    for (int i = 0; i < rows; i++) {

- Compute result for each row independently

**2.2 Initialize sum for row**

    double sum = 0.0;

- Stores final result of row i
- Start from zero

**2.3 Loop over CSR elements of row**

    for (int k = row_ptrs[i]; k < row_ptrs[i + 1]; k++) {

- Only iterate over non-zero elements of row i
- No need to scan full row

**2.4 Multiply and accumulate**

    sum += values[k] * x[col_indices[k]];

- values[k] → matrix value
- col_indices[k] → column position
- x[...] → vector element
- Multiply matrix value with vector element and add to sum

**2.5 Store result**

    y[i] = sum;

- Final result for row i
- Stored in output vector y

### Key Idea:
Instead of performing **O(rows × cols)** operations, the algorithm performs only **O(nnz)** operations, where nnz is the number of non zero elements.

## How to Compile
    gcc challenge.c -lm -o run

`gcc`: GNU Compiler Collection, used to compile C programs

`challenge.c`: The source code file being compiled

`-lm`:
- Links the math library (libm)
- Required for math functions like fmax(), fabs(), etc.
- Without this, linker errors like undefined reference to fmax occur

`-o run`:
- Specifies the output executable name
- Here, the compiled program will be named `run`

## How to Execute
    ./run

## Output:
```
Iter  0 [ 32x 29, density=0.33, nnz= 326]: PASS (Max error: 0.00e+00)
Iter  1 [ 21x 30, density=0.30, nnz= 192]: PASS (Max error: 0.00e+00)
Iter  2 [ 28x 34, density=0.33, nnz= 328]: PASS (Max error: 0.00e+00)
Iter  3 [ 32x  5, density=0.18, nnz=  38]: PASS (Max error: 0.00e+00)
Iter  4 [ 31x 22, density=0.34, nnz= 228]: PASS (Max error: 0.00e+00)
Iter  5 [ 14x 44, density=0.27, nnz= 180]: PASS (Max error: 0.00e+00)
Iter  6 [ 26x 25, density=0.39, nnz= 242]: PASS (Max error: 0.00e+00)
Iter  7 [ 14x 45, density=0.11, nnz=  57]: PASS (Max error: 0.00e+00)
Iter  8 [ 24x  7, density=0.25, nnz=  41]: PASS (Max error: 0.00e+00)
Iter  9 [  9x 25, density=0.09, nnz=  21]: PASS (Max error: 0.00e+00)
Iter 10 [ 15x 13, density=0.22, nnz=  57]: PASS (Max error: 0.00e+00)
Iter 11 [ 23x 22, density=0.37, nnz= 194]: PASS (Max error: 0.00e+00)
Iter 12 [ 19x 14, density=0.38, nnz= 101]: PASS (Max error: 0.00e+00)
Iter 13 [ 18x 30, density=0.18, nnz=  90]: PASS (Max error: 0.00e+00)
Iter 14 [ 23x 26, density=0.39, nnz= 226]: PASS (Max error: 0.00e+00)
Iter 15 [ 29x 16, density=0.08, nnz=  38]: PASS (Max error: 0.00e+00)
Iter 16 [  7x 27, density=0.35, nnz=  60]: PASS (Max error: 0.00e+00)
Iter 17 [ 15x 31, density=0.15, nnz=  66]: PASS (Max error: 0.00e+00)
Iter 18 [ 19x 30, density=0.22, nnz= 109]: PASS (Max error: 0.00e+00)
Iter 19 [ 27x 18, density=0.22, nnz= 107]: PASS (Max error: 0.00e+00)
Iter 20 [ 39x 10, density=0.08, nnz=  29]: PASS (Max error: 0.00e+00)
Iter 21 [ 29x  7, density=0.07, nnz=   4]: PASS (Max error: 0.00e+00)
Iter 22 [ 20x 18, density=0.31, nnz= 100]: PASS (Max error: 0.00e+00)
Iter 23 [ 10x 14, density=0.21, nnz=  26]: PASS (Max error: 0.00e+00)
Iter 24 [ 28x 18, density=0.22, nnz= 117]: PASS (Max error: 0.00e+00)
Iter 25 [ 21x 22, density=0.29, nnz= 126]: PASS (Max error: 0.00e+00)
Iter 26 [ 30x 30, density=0.29, nnz= 271]: PASS (Max error: 0.00e+00)
Iter 27 [ 21x 37, density=0.11, nnz=  79]: PASS (Max error: 0.00e+00)
Iter 28 [ 44x 32, density=0.35, nnz= 495]: PASS (Max error: 0.00e+00)
Iter 29 [ 25x 12, density=0.36, nnz=  96]: PASS (Max error: 0.00e+00)
Iter 30 [ 22x  5, density=0.26, nnz=  27]: PASS (Max error: 0.00e+00)
Iter 31 [  8x 10, density=0.27, nnz=  21]: PASS (Max error: 0.00e+00)
Iter 32 [ 29x 28, density=0.24, nnz= 235]: PASS (Max error: 0.00e+00)
Iter 33 [ 43x 19, density=0.33, nnz= 273]: PASS (Max error: 0.00e+00)
Iter 34 [ 36x 29, density=0.29, nnz= 299]: PASS (Max error: 0.00e+00)
Iter 35 [ 41x 15, density=0.10, nnz=  63]: PASS (Max error: 0.00e+00)
Iter 36 [ 26x 18, density=0.16, nnz=  68]: PASS (Max error: 0.00e+00)
Iter 37 [  5x 25, density=0.27, nnz=  38]: PASS (Max error: 0.00e+00)
Iter 38 [ 12x 11, density=0.16, nnz=  26]: PASS (Max error: 0.00e+00)
Iter 39 [ 45x 11, density=0.05, nnz=  26]: PASS (Max error: 0.00e+00)
Iter 40 [ 45x  8, density=0.24, nnz=  94]: PASS (Max error: 0.00e+00)
Iter 41 [ 36x 18, density=0.20, nnz= 156]: PASS (Max error: 0.00e+00)
Iter 42 [ 26x 44, density=0.21, nnz= 251]: PASS (Max error: 0.00e+00)
Iter 43 [ 32x 29, density=0.31, nnz= 301]: PASS (Max error: 0.00e+00)
Iter 44 [ 10x 28, density=0.38, nnz= 101]: PASS (Max error: 0.00e+00)
Iter 45 [ 19x 34, density=0.15, nnz=  99]: PASS (Max error: 0.00e+00)
Iter 46 [ 11x 41, density=0.27, nnz= 132]: PASS (Max error: 0.00e+00)
Iter 47 [ 31x  6, density=0.26, nnz=  35]: PASS (Max error: 0.00e+00)
Iter 48 [ 20x 34, density=0.28, nnz= 212]: PASS (Max error: 0.00e+00)
Iter 49 [  6x 44, density=0.19, nnz=  59]: PASS (Max error: 0.00e+00)
Iter 50 [ 28x 26, density=0.31, nnz= 237]: PASS (Max error: 0.00e+00)
Iter 51 [ 25x 29, density=0.14, nnz=  95]: PASS (Max error: 0.00e+00)
Iter 52 [ 33x 45, density=0.33, nnz= 466]: PASS (Max error: 0.00e+00)
Iter 53 [ 33x 18, density=0.18, nnz=  90]: PASS (Max error: 0.00e+00)
Iter 54 [ 10x  5, density=0.39, nnz=  18]: PASS (Max error: 0.00e+00)
Iter 55 [ 37x 13, density=0.35, nnz= 177]: PASS (Max error: 0.00e+00)
Iter 56 [  6x 13, density=0.38, nnz=  20]: PASS (Max error: 0.00e+00)
Iter 57 [ 15x 18, density=0.08, nnz=  26]: PASS (Max error: 0.00e+00)
Iter 58 [ 19x 24, density=0.17, nnz=  85]: PASS (Max error: 0.00e+00)
Iter 59 [ 45x 43, density=0.33, nnz= 629]: PASS (Max error: 0.00e+00)
Iter 60 [ 31x 13, density=0.24, nnz= 112]: PASS (Max error: 0.00e+00)
Iter 61 [ 39x 23, density=0.18, nnz= 161]: PASS (Max error: 0.00e+00)
Iter 62 [ 20x 32, density=0.18, nnz= 111]: PASS (Max error: 0.00e+00)
Iter 63 [ 38x 20, density=0.28, nnz= 209]: PASS (Max error: 0.00e+00)
Iter 64 [  9x 34, density=0.22, nnz=  67]: PASS (Max error: 0.00e+00)
Iter 65 [ 29x 31, density=0.15, nnz= 132]: PASS (Max error: 0.00e+00)
Iter 66 [ 38x 24, density=0.18, nnz= 155]: PASS (Max error: 0.00e+00)
Iter 67 [  8x 41, density=0.33, nnz= 103]: PASS (Max error: 0.00e+00)
Iter 68 [ 13x 44, density=0.39, nnz= 221]: PASS (Max error: 0.00e+00)
Iter 69 [ 15x 43, density=0.09, nnz=  61]: PASS (Max error: 0.00e+00)
Iter 70 [ 40x 28, density=0.09, nnz=  98]: PASS (Max error: 0.00e+00)
Iter 71 [ 35x 10, density=0.08, nnz=  25]: PASS (Max error: 0.00e+00)
Iter 72 [ 42x 15, density=0.33, nnz= 230]: PASS (Max error: 0.00e+00)
Iter 73 [ 25x 13, density=0.15, nnz=  37]: PASS (Max error: 0.00e+00)
Iter 74 [ 44x 44, density=0.24, nnz= 435]: PASS (Max error: 0.00e+00)
Iter 75 [ 32x  6, density=0.28, nnz=  57]: PASS (Max error: 0.00e+00)
Iter 76 [ 13x 31, density=0.17, nnz=  60]: PASS (Max error: 0.00e+00)
Iter 77 [ 25x 13, density=0.39, nnz= 126]: PASS (Max error: 0.00e+00)
Iter 78 [ 39x 26, density=0.07, nnz=  70]: PASS (Max error: 0.00e+00)
Iter 79 [ 21x 15, density=0.34, nnz= 100]: PASS (Max error: 0.00e+00)
Iter 80 [ 44x 25, density=0.22, nnz= 250]: PASS (Max error: 0.00e+00)
Iter 81 [ 30x 38, density=0.29, nnz= 324]: PASS (Max error: 0.00e+00)
Iter 82 [ 15x 11, density=0.18, nnz=  41]: PASS (Max error: 0.00e+00)
Iter 83 [ 32x 27, density=0.34, nnz= 289]: PASS (Max error: 0.00e+00)
Iter 84 [ 20x 35, density=0.23, nnz= 159]: PASS (Max error: 0.00e+00)
Iter 85 [ 12x 35, density=0.13, nnz=  60]: PASS (Max error: 0.00e+00)
Iter 86 [ 38x 41, density=0.10, nnz= 145]: PASS (Max error: 0.00e+00)
Iter 87 [ 26x 18, density=0.06, nnz=  30]: PASS (Max error: 0.00e+00)
Iter 88 [ 18x 35, density=0.17, nnz= 114]: PASS (Max error: 0.00e+00)
Iter 89 [ 39x 37, density=0.30, nnz= 405]: PASS (Max error: 0.00e+00)
Iter 90 [ 30x 44, density=0.06, nnz=  70]: PASS (Max error: 0.00e+00)
Iter 91 [ 21x 24, density=0.08, nnz=  42]: PASS (Max error: 0.00e+00)
Iter 92 [ 26x 27, density=0.37, nnz= 250]: PASS (Max error: 0.00e+00)
Iter 93 [ 18x 38, density=0.08, nnz=  52]: PASS (Max error: 0.00e+00)
Iter 94 [ 12x 32, density=0.25, nnz=  93]: PASS (Max error: 0.00e+00)
Iter 95 [ 28x 14, density=0.17, nnz=  57]: PASS (Max error: 0.00e+00)
Iter 96 [  5x 38, density=0.08, nnz=  15]: PASS (Max error: 0.00e+00)
Iter 97 [ 31x 20, density=0.31, nnz= 209]: PASS (Max error: 0.00e+00)
Iter 98 [ 38x 42, density=0.11, nnz= 167]: PASS (Max error: 0.00e+00)
Iter 99 [ 41x  5, density=0.16, nnz=  26]: PASS (Max error: 0.00e+00)

All tests passed! (100/100 iterations passed)
```
## Testing

The implementation is verified using a randomized test harness that ensures correctness across different matrix sizes and sparsity levels.

For each test case, a dense matrix multiplication result is computed as a reference and compared against the CSR based implementation. The comparison uses a floating point tolerance to account for minor numerical precision differences.

All test cases passed successfully, confirming that the CSR implementation produces correct and stable results across varying inputs.
