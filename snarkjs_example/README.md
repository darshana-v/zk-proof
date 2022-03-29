1. Starting a powers of tau ceremony using bn128 curve with 2^12 maximum constraints in the generation of zk-snark paramteres for circuits.

```
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
```

2. Create a ptau file with a new contribution. Enter a random text when prompted.

```
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
```

3. Providing a second contribution while providing a random text as part of the command.

```
snarkjs powersoftau contribute pot12_0001.ptau pot12_0002.ptau --name="Second contribution" -v -e="some random text"
```

4. Providing a third contribution using third party software

```
snarkjs powersoftau export challenge pot12_0002.ptau challenge_0003
snarkjs powersoftau challenge contribute bn128 challenge_0003 response_0003` `-e="some random text"
snarkjs powersoftau import response pot12_0002.ptau response_0003 pot12_0003.ptau -n="Third contribution name"
```

5. Verifying the protocol - checking all the contributions to the multi-party computation (MPC) up to that point, prints hashes of all the intermediate results to that console.

```
snarkjs powersoftau verify pot12_0003.ptau
```

6. Applying a random beacon to finalise phase 1 of the trusted setup

```
snarkjs powersoftau beacon pot12_0003.ptau pot12_beacon.ptau 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon"
```

7. Preparing phase 2 of the setup - calculating the encrypted evaluation of the Lagrange polynomials at tau for `tau`, `alpha*tau` and `beta*tau` by taking the beacon ptau file generated and producing a final ptau file as output which will be used to generate the circuit proving and verification keys.

```
snarkjs powersoftau prepare phase2 pot12_beacon.ptau pot12_final.ptau -v
```

8. Verifying the final ptau protocol transcript

```
snarkjs powersoftau verify pot12_final.ptau
```

9. Creating the circuit

```
 cat <<EOT > circuit.circom
 template Multiplier(n) {
    signal input a;
    signal input b;
    signal output c;

    signal int[n];

    int[0] <== a*a + b;
    for(var i=1; i<n; i++) {
        int[i] <== int[i-1]*int[i-1] + b;
    }
    c <== int[n-1];

}

 component main = Multiplier(1000);
 EOT
```

10. Compiling circuit

```
circom circuit.circom --r1cs --wasm --sym
```

11. View info about the circuit

```
snarkjs r1cs info circuit.r1cs
```

12. Printing constraints

```
snarkjs r1cs print circuit.r1cs circuit.sym
```

13. Exporting r1cs to json

```
snarkjs r1cs export json circuit.r1cs circuit.r1cs.json cat circuit.r1cs.json
```

14. Calculating witness

```
cat <<EOT > input.json {"a": 3, "b": 11} EOT
snarkjs calculatewitness --wasm circuit.wasm --input input.json
```

15. Setting up groth16 proving system

```
snarkjs groth16 setup circuit.r1cs pot12_final.ptau circuit_final.zkey
```

16. Contributing to phase 2 ceremony by using "zkey contribute" to create a zkey file with a new contribution, enter random text when prompted

```
snarkjs zkey contribute circuit_final.zkey circuit_final_1.zkey --name="1st Contributor Name" -v
```

17. Contributing to the phase 2 ceremony

```
snarkjs zkey contribute circuit_final_1.zkey circuit_final_2.zkey --name="1st Contributor Name" -v
```

18. Providing third contribution using third party software

```
snarkjs zkey export bellman circuit_final_2.zkey challenge_phase2_0003 snarkjs zkey bellman contribute bn128 challenge_phase2_0003 response_phase2_0003 -e="some random text" snarkjs zkey import bellman circuit_final_2.zkey response_phase2_0003 circuit_final_3.zkey -n="Third contribution name"
```

19. Verifying the latest zkey

```
snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_final_3.zkey
```

20. Applying random beacon

```
snarkjs zkey beacon circuit_final_3.zkey circuit_final_key.zkey 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon phase2"
```

21. Verifying final zkey

```
snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_final_key.zkey
```

22. Exporting verifiation key

```
snarkjs zkey export verificationkey circuit_final_key.zkey verification_key.json
```

23. Creating groth16 proof

```
snarkjs groth16 prove circuit_final_key.zkey witness.wtns proof.json public.json
```

Alternately, creating proof and calculating witness can be done together by:

```
snarkjs groth16 fullprove input.json circuit.wasm circuit_final_key.zkey proof.json public.json
```

24. Verifying the proof

```
snarkjs groth16 verify verification_key.json public.json proof.json
```

25. Turning the verifier into a smart contract

```
snarkjs zkey export solidityverifier circuit_final_key.zkey verifier.sol
```

26. Simulating a verification call

```
snarkjs zkey export soliditycalldata public.json proof.json
```

The following result is obtained:
`["0x1c128afe18c710ccfe5bf25a958c4553392139cb7f80877419dfe13be2164bb5", "0x014aac69a503e99bca30aab66da9331fbc995999b680544ec28ecc4b77178af7"],[["0x2f4e2430551286dba691180e2996cb32fc5be2dbd350809e1bc251a32491f2f1", "0x1b966a84c23cb605b7a6f0c5c4958701af1609639c2b25390b9edb680403dbc3"],["0x03f1c98883825342cca0f1b7de5f632488e526b6a9bb632c94b41b51cc277350", "0x0f6516ead3d3f7e9dc571d0a1014fb37a26eb2e25446eec073ec6499c4254608"]],["0x0e2ec1558fa7245af197cb61b1f882afc3dcb878cc7a91595b47946f225a52d0", "0x1e9780c54c07423cedd3d73f0571db8f0fe580ccc4cf8c742cc8da633c1bb886"],["0x110d778eaf8b8ef7ac10f8ac239a14df0eb292a8d1b71340d527b26301a9ab08","0x0000000000000000000000000000000000000000000000000000000000000003","0x000000000000000000000000000000000000000000000000000000000000000b"]`

Cut and paste the result directly in the verifyProof field in the deployed smart contract
