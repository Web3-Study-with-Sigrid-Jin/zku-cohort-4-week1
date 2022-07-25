# Boom Labs - Week 1 ZKU

### Part 1

#### Explain in 2-4 sentences why SNARK requires a trusted setup while STARK doesn’t.

* zkSTARKs requires a trusted setup because it requires an initial creation event of the keys, which are used to create proofs for private transactions, and used in the verification of those proofs.
* On the other hand, zkSTARKs does not require a trusted-setup for utilizing the network because it is transparent which means that they use publicly verifiable randomness to create a trustless verifiable computing system.

#### Name two more differences between SNARK and STARK proofs.

* zkSTARKs rely on hash functions, so it would be a quantum resistant, on the other hand zkSNARKs are not quantum resistant so the privacy of SNARKs could be broken. SNARKs rely on a [common reference string](https://medium.com/coinplug-tech/tagged/common-reference-string) (CRS)
  * What is a common reference string?
    * For Non-Interactive Zero-Knowledge Proof System (NIZK), there is one convention between the attestant and the validator.
    * In the Interactive, random values are exchanged to prevent cheating the other party, but non-interactive means that if the prover generates a proof, the verifier verifies it although they do not exchange additional messages when verifying the statement.
    * The value for this is Common Reference String (CRS) which can be a pure random values method, or a random value associated with the content you want to prove. It is this CRS that many people think is the key to zero-knowledge proof, and this CRS is used by both the attestant and the validator.
* zkSTARKs have larger proof sizes than zkSNARKs, which means that the verification in STARKs takes more time than in SNARKs and SNARKs are much gas efficient.
* STARK can handle larger proofs sizes than SNARK ( so it needs more time and consequently more gas, STARK came years after SNARK and that's why it has lesser adoption.

#### What is a Powers of Tau ceremony? Explain why this is important in the setup of zk-SNARK applications.

* Powers of Tau is a ceremony where multiple people creates the setup keys for SNARK, with a key someone could change values and fake transactions leaving the network insecure. By doing this type of ceremony the key is usually destroyed guaranteeing that the network is secure.-

### Part 2

Fork the `Week 1` repo and go into the `Q2` directory. Install all the node dependencies. In the `contracts/circuits` folder, you will find `HelloWorld.circom`. Run the bash script `scripts/compile-HelloWorld.sh` to compile the circuit.

#### What does the circuit in `HelloWorld.circom` do ?

```
/*This circuit template checks that c is the multiplication of a and b.*/

template Multiplier2 () {

   // Declaration of signals.
   signal input a;
   signal input b;
   signal output c;

   // Constraints.
   c <== a * b;
}

component main = Multiplier2();
```

It implements a circom circuit to prove the multiplication of two private inputs signal identifiers `a & b`, and calculate the result in a public output signal identifier `c`.

#### Lines 7-12 of `compile-HelloWorld.sh` download a file called `powersOfTau28_hez_final_10.ptau` for Phase 1 trusted setup.

```
if [ -f ./powersOfTau28_hez_final_10.ptau ]; then
    echo "powersOfTau28_hez_final_10.ptau already exists. Skipping."
else
    echo 'Downloading powersOfTau28_hez_final_10.ptau'
    wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_10.ptau
fi
```

* The power of tau ceremony is to secure multi-party computation MPC ceremony. The stage is used to generate the parameters of first phase in zk-SNARKs which consists of two phases. The MPC ceremony can generate parameters for all the circuits up to `2^21`.  It proceeds in turns, one turn for each party, and the result of the computation of each party is then added to a public transcript, which will allow all the system be verified. The result parameters of the system then becomes secure if one party was able to destroy the random toxic waste.
* Explain why this is important in the setup of zk-SNARK applications?
  * The first phase creates the randomness for the ceremony. In order to deploy a zkSNARK circuits, a developer must perform a computation for generating the "Proving Key", and the "Verifying key", and this process called a "Trusted Setup".
  * There is a downside for this process because it produces bad numbers called "toxic waste" file that needs to be destroyed otherwise it will produce fake proofs which will violate the security of the system.
  * So to resolve that, the "Trusted Setup" can be setup using specific Cryptographic ceremony like the "Powers of Tau" ceremony, which will be used only in generating the first phase of parameter generation of all the projects, since zk-SNARKs requires two phases of parameter generation.
  * Power of Tau ceremony is helping in creating what is called a "Chain of trusted-setup", through allowing multiple parties to setup a trusted setup and then adding the results to a public transcript, and the system can be verified by publishing the results. Moreover, the system will be considered secure, if only one of the parties has successfully destroying the "toxic waste" file.

#### Line 24 of `compile-HelloWorld.sh` makes a random entropy contribution as a Phase 2 trusted setup. How are Phase 1 and Phase 2 trusted setup ceremonies different from each other?

```
snarkjs zkey contribute HelloWorld/circuit_0000.zkey HelloWorld/circuit_final.zkey --name="1st Contributor Name" -v -e="random text"
```

​		zkSNARK projects consists of two phases of parameter generation, The Power of Tau can be only used in the first phase for all the projects. While the second phase if a circuit-speicfic that needs to be done for each circuit, and the individual teams are responsible on it. 

​		Phase-one is a universal phase ceremony for the entire community, while to start phase-two, any zkSNARK projects can pick any point of the ceremony to begin their circuit-specific second phase.

#### In the empty `scripts/compile-Multiplier3-groth16.sh`, create a script to compile `contracts/circuits/Multiplier3.circom` and create a verifier contract modeling after `compile-HelloWorld.sh`     

```bash
# Compile circuit

circom Multiplier3.circom --r1cs --wasm --sym -o Multiplier3
snarkjs r1cs info Multiplier3/Multiplier3.r1cs

# Start a new zkey and make a contribution
#!/bin/bash

# [assignment] create your own bash script to compile Multiplier3.circom modeling after compile-HelloWorld.sh below

cd contracts/circuits

mkdir Multiplier3

if [ -f ./powersOfTau28_hez_final_10.ptau ]; then
    echo "powersOfTau28_hez_final_10.ptau already exists. Skipping."
else
    echo 'Downloading powersOfTau28_hez_final_10.ptau'
    wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_10.ptau
fi

echo "Compiling Multiplier3.circom..."

# Compile circuit

circom Multiplier3.circom --r1cs --wasm --sym -o Multiplier3
snarkjs r1cs info Multiplier3/Multiplier3.r1cs

# Start a new zkey and make a contribution

snarkjs groth16 setup Multiplier3/Multiplier3.r1cs powersOfTau28_hez_final_10.ptau Multiplier3/circuit_0000.zkey
snarkjs zkey contribute Multiplier3/circuit_0000.zkey Multiplier3/circuit_final.zkey --name="1st Contributor Name" -v -e="boom labs"
snarkjs zkey export verificationkey Multiplier3/circuit_final.zkey Multiplier3/verification_key.json

# generate solidity contract
snarkjs zkey export solidityverifier Multiplier3/circuit_final.zkey ../Multiplier3Verifier.sol

cd ../..
```

#### Try to run `compile-Multiplier3-groth16.sh`. You should encounter an `error[T3001]` with the circuit as is. Explain what the error means and how it arises.

```bash
error[T3001]: Non quadratic constraints are not allowed!
   ┌─ "../contracts/circuits/Multiplier3.circom":14:4
   │
14 │    d <== a * b * c;
   │    ^^^^^^^^^^^^^^^ found here
   │
   = call trace:
     ->Multiplier3
```

The error comes with `Non Quandartic Constraints...` arises because the circuit contains the constraint `d <== a * b * c` is cubic instead of quadratic. Circom **only** allows for quadratic constraints.

#### Modify `Multiplier3.circom` to perform a multiplication of three input signals under the restrictions of circom.

```lisp
pragma circom 2.0.0;

template Multiplier3 () {  

   // Declaration of signals.  
   signal input a;  
   signal input b;
   signal input c;
	 signal intermediateMultiplier; // holds the intermediate multiplication value
   signal output d;  

	intermediateMultiplier <== a * b; // computing intermediate value

   // Constraints.
   d <== intermediateMultiplier * c; // computing final value
}

component main = Multiplier3();
```

#### In the empty `scripts/compile-Multiplier3-plonk.sh`, create a script to compile `circuit/Multiplier3.circom` using PLONK in snarkjs. Add a `_plonk` suffix to the build folder and the output contract to distinguish the two sets of output.

```bash
#!/bin/bash

# [assignment] create your own bash script to compile Multiplier3.circom using PLONK below

cd contracts/circuits

mkdir Multiplier3_PLONK

if [ -f ./powersOfTau28_hez_final_10.ptau ]; then
    echo "powersOfTau28_hez_final_10.ptau already exists. Skipping."
else
    echo 'Downloading powersOfTau28_hez_final_10.ptau'
    wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_10.ptau
fi

echo "Compiling Multiplier3.circom..."

# Compile the circuit

circom ../contracts/circuits/Multiplier3.circom --r1cs --wasm --sym -o Multiplier3_PLONK
snarkjs r1cs info Multiplier3_PLONK/Multiplier3.r1cs

echo "Using snarkjs to create zkey..."

# Start a new zkey and make a contribution

snarkjs plonk setup Multiplier3_PLONK/Multiplier3.r1cs powersOfTau28_hez_final_10.ptau Multiplier3_PLONK/circuit_final.zkey

# No this phase: snarkjs zkey contribute Multiplier3/circuit_0000.zkey Multiplier3/circuit_final.zkey --name="1st Contributor Name" -v -e="boom labs"

snarkjs zkey export verificationkey Multiplier3_PLONK/circuit_final.zkey Multiplier3_PLONK/verification_key.json

echo "Generating solidity contract..."

# Generate solidity contract
snarkjs zkey export solidityverifier Multiplier3_PLONK/circuit_final.zkey ../Multiplier3PlonkVerifier.sol

cd ../..
```

1. How is the process of compiling with PLONK different from compiling with Groth16?

* PLONK does not require a ceremony unlike Groth16. The contribute, which is second step is not needed and we can just directly export the verfication key.

2. The practical differences between Groth16 and PLONK is what?

* Groth16 requires a trusted ceremony to each circuit, while PLONK does not require it, its just enough to use The Power of Tau universal setup ceremony
* After running the test in Question (5), it turns out that Groth16 proofing system took shorter time than PLONK proofing system, where Groth16 took 1132ms, and PLONK took 1462ms.

#### So far we have not tested our circuit yet. While you can verify your circuit in the terminal using `snarkjs groth16 fullprove`, you can also do so directly in a Node.js script. We will practice doing so by creating some unit tests to try out our verifier contract(s):

* using regex to update the solidity version declaration and differentiate each contract verifiers.

![image-20220725163637023](https://user-images.githubusercontent.com/41055141/180749050-f1538e36-9d93-47c1-b83d-5eac9b88f85a.png)

```javascript
const { expect, assert } = require('chai')
const { ethers } = require('hardhat')
const { groth16, plonk } = require('snarkjs')

const wasm_tester = require('circom_tester').wasm

const F1Field = require('ffjavascript').F1Field
const Scalar = require('ffjavascript').Scalar
exports.p = Scalar.fromString(
  '21888242871839275222246405745257275088548364400416034343698204186575808495617',
)
const Fr = new F1Field(exports.p)

describe('HelloWorld', function () {
  this.timeout(100000000)
  let Verifier
  let verifier

  beforeEach(async function () {
    Verifier = await ethers.getContractFactory('HelloWorldVerifier')
    verifier = await Verifier.deploy()
    await verifier.deployed()
  })

  it('Circuit should multiply two numbers correctly', async function () {
    // circuit을 import하는데, 이 때 이용하는 용도가 wasm_tester라는 모듈이다
    const circuit = await wasm_tester('contracts/circuits/HelloWorld.circom')

    // 사용자의 input을 정의한다
    const INPUT = {
      a: 2,
      b: 3,
    }

    // witness를 계산하는데, circuit과 이전 input을 사용하여 계산한다.
    const witness = await circuit.calculateWitness(INPUT, true)

    console.log(witness)

    // assert claim!
    // circuit의 output이 a * b와 같은 지 확인하기 위함
    assert(Fr.eq(Fr.e(witness[0]), Fr.e(1)))
    assert(Fr.eq(Fr.e(witness[1]), Fr.e(6)))
  })

  it('Should return true for correct proof', async function () {
    // generate proof and circuit ouputs (public signals) using inputs (a & b), witness and keys
    const { proof, publicSignals } = await groth16.fullProve(
      { a: '2', b: '3' },
      'contracts/circuits/HelloWorld/HelloWorld_js/HelloWorld.wasm',
      'contracts/circuits/HelloWorld/circuit_final.zkey',
    )

    // log first output public signal
    console.log('2x3 =', publicSignals[0])

    // generates parameters for solidity verifier
    const calldata = await groth16.exportSolidityCallData(proof, publicSignals)

    // split calldata variable to array containing proof and inputs to be passed
    const argv = calldata
      .replace(/["[\]\s]/g, '')
      .split(',')
      .map((x) => BigInt(x).toString())

    // assign parameters to their respective variables
    const a = [argv[0], argv[1]]
    const b = [
      [argv[2], argv[3]],
      [argv[4], argv[5]],
    ]
    const c = [argv[6], argv[7]]
    const Input = argv.slice(8)

    // call solidity verifier and veify the proof of a * b = c
    expect(await verifier.verifyProof(a, b, c, Input)).to.be.true
  })
});

describe("Multiplier3 with Groth16", function () {
	let Verifier;
	let verifier;

	beforeEach(async function () {
		Verifier = await ethers.getContractFactory("Multiplier3Verifier");
		verifier = await Verifier.deploy();
		await verifier.deployed();
	});

	it("Circuit should multiply three numbers correctly", async function () {
		const circuit = await wasm_tester("contracts/circuits/Multiplier3.circom");

		const INPUT = {
			a: 2,
			b: 3,
			c: 4,
		};

		const witness = await circuit.calculateWitness(INPUT, true);

		console.log(witness);

		assert(Fr.eq(Fr.e(witness[0]), Fr.e(1)));
		assert(Fr.eq(Fr.e(witness[1]), Fr.e(24)));
	});

	it("Should return true for correct proof", async function () {
		// procedure already explained in comments above for Multiplier2 / HelloWorld circuit
		const { proof, publicSignals } = await groth16.fullProve(
			{ a: "2", b: "3", c: "4" },
			"contracts/circuits/Multiplier3/Multiplier3_js/Multiplier3.wasm",
			"contracts/circuits/Multiplier3/circuit_final.zkey"
		);

		console.log("2 * 3 * 4 =", publicSignals[0]);

		const calldata = await groth16.exportSolidityCallData(proof, publicSignals);

		const argv = calldata
			.replace(/["[\]\s]/g, "")
			.split(",")
			.map((x) => BigInt(x).toString());

		const a = [argv[0], argv[1]];
		const b = [
			[argv[2], argv[3]],
			[argv[4], argv[5]],
		];
		const c = [argv[6], argv[7]];
		const Input = argv.slice(8);

		expect(await verifier.verifyProof(a, b, c, Input)).to.be.true;
	});

	it("Should return false for invalid proof", async function () {
		let a = [0, 0];
		let b = [
			[0, 0],
			[0, 0],
		];
		let c = [0, 0];
		let d = [0];

		expect(await verifier.verifyProof(a, b, c, d)).to.be.false;
	});
});

describe("Multiplier3 with PLONK", function () {
	let Verifier;
	let verifier;

	beforeEach(async function () {
		Verifier = await ethers.getContractFactory("PlonkVerifier");
		verifier = await Verifier.deploy();
		await verifier.deployed();
	});

	it("Should return true for correct proof", async function () {
		const { proof, publicSignals } = await plonk.fullProve(
			{ a: "2", b: "3", c: "4" },
			"contracts/circuits/Multiplier3_PLONK/Multiplier3_js/Multiplier3.wasm",
			"contracts/circuits/Multiplier3_PLONK/circuit_final.zkey"
		);

		console.log("2 * 3 * 4 =", publicSignals[0]);

		const calldata = await plonk.exportSolidityCallData(proof, publicSignals);
		const calldataProof = calldata.split(",")[0];

		expect(await verifier.verifyProof(calldataProof, publicSignals)).to.be.true;
	});

	it("Should return false for invalid proof", async function () {
		let a = [0, 0];
		let b = [0];

		expect(await verifier.verifyProof(a, b)).to.be.false;
	});
});
```

![image-20220725155839813](https://user-images.githubusercontent.com/41055141/180749059-5167ed30-c958-465e-ac60-972156f9095d.png)

1. 

### Part 3

#### [circomlib](https://github.com/iden3/circomlib) is the official library of circuit templates released by iden3, the creator of Circom. One important template included is `comparators.circom`, which implements value comparisons between two numbers. The following questions will cover the use of this template in our own circuits:

```lisp
// Circuit to prove a number is less than 10
// https://gist.github.com/ChubbyCub/b07c096924d9a4cd55d3c57dc669aeff

pragma circom 2.0.0;
include "./node_modules/circomlib/circuits/comparators.circom";

/* Wrapper template around the existing circomlib template LessThan.
    @param n: needs to at least the max bit count between input a and the number 10.
    We can assign n to be 64-bit as it is generic enough for an integer input.
    The example below demonstrates the logic behind the circuit:

    Signal input a = 15, which can be represented with 4 bits (1 1 1 1). 
    The LessThan circuit attempts to build the bit representation of the minimum integer that requires 
    one (1) additional bit, which is 16 (represented with 5 bits: 1 0 0 0 0)
    The LessThan circuit will then add the result of (a - 10) to the 5-bit number. 
    If a - 10 < 0, then the most significant bit of 5-bit number will become 0.
    If a - 10 > 0, then the most significant bit of 5-bit number will remain 1.
    Signal output = 1 - the most significant bit of 5-bit number, 
    which is either 0 (meaning a >= 10) or 1 (meaning a < 10).
    // 그냥 해주면 되지 왜 이렇게 만들었지...
*/

template ex2(n) {
    signal input a;
    signal output out;
    var x = 10;

    component lessThan = LessThan(n);

    lessThan.in[0] <-- a;
    lessThan.in[1] <-- x;

    out <== lessThan.out;
}

// The 64 or 32 is the number of bits that we expect as input for the LessThan.
component main = ex2(64);
```

* `n` variable in the `template LessThan(n)` function refers to the number of bits for the input signal, and `32` stands for the unsigned integers, non-negative integer in the range `0 to 4294967295`.

#### What are the possible outputs for the `LessThan` template and what do they mean respectively? (If you cannot figure this out by reading the code alone, feel free to compile the circuit and test with different input values.)

* The expected outputs are either `0` or `1` which is a boolean return datatype, so if the number `n1` is not less than number `n2` the output will be `0` which means false `n1 > n2`. Otherwise the output will be `1` which means true `n1 < n2`.

#### Proving a number is within a range without revealing the actual number could be useful in applications like proving our income when applying for a credit card. In `contracts/circuits/RangeProof.circom`, create a template (not circuit, so don’t add `component main = ...`) that uses `GreaterEqThan` and `LessEqThan` to perform a range proof.

```lisp
pragma circom 2.0.0;

include "../../node_modules/circomlib/circuits/comparators.circom";

template RangeProof(n) {
    assert(n <= 252);
    signal input in; // this is the number to be proved inside the range
    signal input range[2]; // the two elements should be the range, i.e. [lower bound, upper bound]
    signal output out;

    component lt = LessEqThan(n);
    component gt = GreaterEqThan(n);

    // upper bound
    
    lt.in[0] <== in;
    lt.in[1] <== range[1];

    // lower bound

    gt.in[0] <== in;
    gt.in[1] <== range[0];

    out <== lt.out * gt.out;
}
```

#### [circomlib-matrix](https://github.com/socathie/circomlib-matrix) is a library covering basic matrix operations, modeled after circomlib, and created by our very own mentor Cathie. Matrix operations can be useful in puzzles (e.g. [zkPuzzles](https://github.com/zku-cohort-3/zkPuzzles), [zkGames](https://github.com/vplasencia/zkGames)), image processing (e.g [zkPhoto](https://github.com/socathie/zkPhoto)), and machine learning (e.g. [zk-mnist](https://github.com/0xZKML/zk-mnist), [zk-ml](https://github.com/zk-ml/demo)). Let’s take a look at matrix operations in action in a [Sudoku](https://en.wikipedia.org/wiki/Sudoku) circuit in the [zkPuzzles](https://github.com/zku-cohort-3/zkPuzzles) repo.

##### In `projects/zkPuzzles/circuits`, modify Lines 20-23 of `sudoku.circom` so that it implements the check on the inputs to be between 0 and 9 (inclusive) using your `RangeProof` template from 1.3.

(As-is)

```
//[assignment] hint: you will need to initialize your RangeProof components here
    
    for (var i=0; i<9; i++) {
        for (var j=0; j<9; j++) {
            assert(puzzle[i][j]>=0); //[assignment] change assert() to use your created RangeProof instead
            assert(puzzle[i][j]<=9); //[assignment] change assert() to use your created RangeProof instead
            assert(solution[i][j]>=0); //[assignment] change assert() to use your created RangeProof instead
            assert(solution[i][j]<=9); //[assignment] change assert() to use your created RangeProof instead
            mul.a[i][j] <== puzzle[i][j];
            mul.b[i][j] <== solution[i][j];
        }
    }
    for (var i=0; i<9; i++) {
        for (var j=0; j<9; j++) {
            mul.out[i][j] === 0;
        }
    }
```

(To-be)

```
component puzzzleRange[9][9] ;
    component solutionRange[9][9] ;


    for (var i=0; i<9; i++) {
        for (var j=0; j<9; j++) {
           puzzzleRange[i][j] = RangeProof(32);
            puzzzleRange[i][j].in <== puzzle[i][j];
            puzzzleRange[i][j].range[0] <== 0;
            puzzzleRange[i][j].range[1] <== 9;
            solutionRange[i][j] = RangeProof(32);
            solutionRange[i][j].in <== solution[i][j];
            solutionRange[i][j].range[0] <== 0;
            solutionRange[i][j].range[1] <== 9;
            puzzzleRange[i][j].out * solutionRange[i][j].out ===1; 
            mul.a[i][j] <== puzzle[i][j];
            mul.b[i][j] <== solution[i][j];
        }
    }
    for (var i=0; i<9; i++) {
        for (var j=0; j<9; j++) {
            mul.out[i][j] === 0;
        }
    }
```

#### You can run `npm run test:fullProof` while inside the `zkPuzzles` directory to test your modified circuit. You are expected to encounter an error. Record the error, resolve it by modifying `project/zkPuzzles/scripts/compile-circuits.sh`, and explain why it has occurred and what you did to solve the error.

![image-20220725175302018](https://user-images.githubusercontent.com/41055141/180749085-8cd3cf22-d566-436f-b230-c8468173b350.png)

The error was snarkJS: circuit too big for this power of tau ceremony. `97588 > 2^16`.

It means that the current circuit requires a bigger power of tau ceremony, because the current one is a `16` ceremony `2^16 < 97588`, which is less than the expected from the circuit `97588`. To resolve this error, just change the downloaded Power of Tau ceremony file to be bigger than `16`, so I downloaded Power of Tau with the power of `20` therefore, `2^20 > 97588`.

```bash
# if [ -f ./powersOfTau28_hez_final_10.ptau ]; then
#     echo "powersOfTau28_hez_final_10.ptau already exists. Skipping."
# else
#     echo 'Downloading powersOfTau28_hez_final_10.ptau'
#     wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_10.ptau
# fi

# ./zku/week1/Q3/projects/zkPuzzles/scripts
if [ -f ./powersOfTau28_hez_final_20.ptau ]; then
    echo "powersOfTau28_hez_final_20.ptau already exists. Skipping."
else
    echo 'Downloading powersOfTau28_hez_final_20.ptau'
    wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_20.ptau
fi
```

```bash
# zku-cohort-4-week1/Q3/scripts
chmod u+x bonus-compile.sh
./bonus-compile.sh
```



1. 
