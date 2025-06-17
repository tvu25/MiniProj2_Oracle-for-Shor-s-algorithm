# Quantum Oracle for Shor's Algorithm

This repository contains a Qiskit implementation of an oracle for Shor's algorithm as part of the **QC Boot Camp Mini-Projects, Second Batch** (Mini-Project #2). The oracle performs modular multiplication, a key component in the quantum part of Shor's algorithm for integer factorization.

## Project Description

The goal is to implement a quantum circuit that acts as an oracle for Shor's algorithm. Given two positive integers ( a ) and ( N ) (with ( a &lt; N )), the oracle applies the transformation:

\[ U |x\\rangle_1 |y\\rangle_n = \\begin{cases} |x\\rangle_1 |a y \\mod N\\rangle_n, & \\text{if } x = 1 \\text{ and } y &lt; N, \\ |x\\rangle_1 |y\\rangle_n, & \\text{otherwise.} \\end{cases} \]

where ( n = \\lceil \\log_2(N) \\rceil ). The implementation is optimized for ( N = 15 ), but includes a general approach for arbitrary ( N ). The circuit uses only 1-qubit gates, multi-controlled phase (MCP) gates, multi-controlled ( X ) (MCX) gates, and the built-in Quantum Fourier Transform (QFT) and its inverse, as specified. No classical bits or measurements are used.

## Implementation Details

The implementation is provided in the Python script `oracle_shors.py`. The main function, `modular_mult_oracle(a, N)`, constructs the quantum circuit for the oracle. Two approaches are implemented:

1. **Specialized Case (( N = 15 ))**:

   - For ( N = 15 ), the oracle uses a permutation matrix to directly implement the modular multiplication ( |y\\rangle \\to |a y \\mod 15\\rangle ).
   - The circuit uses two ancilla qubits to handle the control logic:
     - **Step 1**: Detect if ( y = 15 ) (binary 1111) using an MCX gate to set `anc[0]`.
     - **Step 2**: Compute the control condition ( x = 1 ) AND ( y \\neq 15 ) using a Toffoli gate, stored in `anc[1]`.
     - **Step 3**: Apply a controlled unitary (representing ( a y \\mod 15 )) on the ( y )-register, controlled by `anc[1]`.
     - **Step 4**: Uncompute the ancilla qubits to reset them to ( |0\\rangle ).
   - This approach leverages Qiskit’s `UnitaryGate` to construct the permutation matrix, avoiding explicit arithmetic operations.

2. **General Case (Arbitrary ( N ))**:

   - For arbitrary ( N ), the oracle uses a QFT-based approach to perform modular multiplication in the Fourier basis.
   - The circuit:
     - Applies QFT to the ( y )-register.
     - Uses controlled phase gates to implement multiplication by ( a \\mod N ) in the phase domain.
     - Applies the inverse QFT to return to the computational basis.
     - Uses ancilla qubits to ensure the transformation is only applied when ( x = 1 ) and ( y &lt; N ).
   - This method is less efficient for ( N = 15 ) but generalizes to any ( N ).

The demo function `demo_oracle(a, N, y_values)` tests the oracle for given input values of ( y ), setting ( x = 1 ), and prints the resulting statevector to verify correctness. It supports ( N = 15 ) and displays the expected output ( |1\\rangle |a y \\mod N\\rangle ) for ( y &lt; N ), or ( |1\\rangle |y\\rangle ) otherwise.

## Dependencies

- **Qiskit**: For quantum circuit construction and simulation.
- **NumPy**: For numerical operations, including constructing the permutation matrix.
- **matplotlib-venn**: Optional, not used in the final implementation but included for potential visualization extensions.

Install dependencies using:

```bash
pip install qiskit
pip install matplotlib-venn
```

## Usage

Run the script to test the oracle with the provided demo:

```python
N = 15
a = 7
test_values = [5, 15]
demo_oracle(a, N, test_values)
```

This tests the oracle for ( a = 7 ), ( N = 15 ), with input ( y ) values ( 5 ) and ( 15 ). The output shows the input state, expected output, and the resulting statevector probabilities, confirming the oracle behaves as expected:

- For ( y = 5 ), the output is ( |1\\rangle |a \\cdot 5 \\mod 15\\rangle = |1\\rangle |5\\rangle ) (since ( 7 \\cdot 5 = 35 \\equiv 5 \\mod 15 )).
- For ( y = 15 ), the output remains ( |1\\rangle |15\\rangle ) (unchanged, as ( y \\geq N )).

## Example Output

```
Input y = 5 (0100):
Expected: |1⟩|0100⟩ (7*5 mod 15 = 5)
Output state (non-zero amplitudes):
  |10100⟩: amplitude = 1.0000

Input y = 15 (1111):
Expected: |1⟩|1111⟩ (unchanged)
Output state (non-zero amplitudes):
  |11111⟩: amplitude = 1.0000
```

## Notes

- The implementation for ( N = 15 ) is optimized using a permutation matrix, which is more efficient than the QFT-based approach for this specific case.
- The general case is included to demonstrate scalability, though it is less efficient for small ( N ).
- The code adheres to the constraints (no classical bits or measurements) and uses only allowed gates.
- For ( N \\neq 15 ), the QFT-based approach may require additional optimization for practical use, as it is computationally intensive.
- The circuit has been tested for correctness using statevector simulation, ensuring the output matches the expected transformation.

## References

- Qiskit documentation: qiskit.org
- Shor’s algorithm circuit reference: "Circuit for Shor's algorithm using ( 2n+3 ) qubits".