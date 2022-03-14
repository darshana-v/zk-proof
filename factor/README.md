#### 1. generate circom circuit with given code

#### 2. compile the circuit:

`circom circuit.circom --r1cs --wasm --sym`
- --r1cs: generates r1cs constraint system of the circuit in binary format
- --wasm: generates wasm code to generate the witness
- --sym: generates symbols file required for debugging and printing the constraint system in an annotated mode

#### 3. take the compiled circuit to snarkjs

check if information fits with the mental map of the circuit:

`snarkjs info -r circuit.r1cs`

print constraints of circuit:

`snarkjs printconstraints -r circuit.r1cs -s circuit.sym`

(doesn't work so do the next one:)

`snarkjs rp circuit.r1cs circuit.sym`

Output:
```
[INFO] snarkJS: [ 21888242871839275222246405745257275088548364400416034343698204186575808495616main.a ] \* [ main.b ] - [ 21888242871839275222246405745257275088548364400416034343698204186575808495616main.c ] = 0
```
