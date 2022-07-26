# MUX Design Verification

The verification environment is setup using [Vyoma's UpTickPro](https://vyomasystems.com) provided for the hackathon.

![Screenshot (133)](https://user-images.githubusercontent.com/105343698/182012523-35921775-b37c-4f47-824b-eb3084a278ab.png)

## Verification Environment

The [CoCoTb](https://www.cocotb.org/) based Python test is developed as explained. The test drives inputs to the Design Under Test (mux module here) which takes in 2-bit inputs and 5-bit ``sel``

The values are assigned to the input port using 
```
    A=1
    B=2
    inp_sel=13 

    dut.inp12.value=A
    dut.inp13.value=B
    dut.sel.value=inp_sel
```

The assert statement is used for comparing the mux's outut to the expected value.

The following error is seen:
```
assert dut.out.value == B, "Test failed, because selected input was inp13 and expected output was {B} but the output from DUT is {OUT} ".format(B=dut.inp13.value, OUT=dut.out.value)
```
## Test Scenario **(Important)**
- Test Inputs: A=1 B=2 inp_sel = 13
- Expected Output: EXPECT=002
- Observed Output in the DUT DUT=00001

Output mismatches for the above inputs proving that there is a design bug

## Design Bug
Based on the above test input and analysing the design, we see the following

```
      5'b01101: out = inp12;    ===>BUG
      5'b01101: out = inp13;    
```
For the mux design, the logic should be ``5'b01100`` as ``sel`` value for ``inp12`` instead of ``5'b01101`` as in the design code.

## Test Scenario **(Important)**
- Test Inputs: C=1 inp_sel=30
- Expected Output: EXPECT=001
- Observed Output in the DUT DUT=00000

Output mismatches for the above inputs proving that there is a design bug

## Design Bug
Based on the above test input and analysing the design, we see the following

```
      5'b11101: out = inp29;
      default: out = 0;         ====>BUG
```
For the mux design, the ``sel`` and ``inp30`` are missing in the design code.

## Design Fix
Updating the design and re-running the test makes the test pass.

![Screenshot (134)](https://user-images.githubusercontent.com/105343698/182012534-927fc672-a5e1-4928-8b89-9ef5a276755e.png)

The updated design is checked in as mux_fix.v

## Verification Strategy

Wrote a test ``test_mux``  that checks all values with random inputs. If there are any bugs that werent fixed, they will be detected immediately.
The  fixed design didnt declare any errors after running the final test which verifies that the updated design is working properly.

## Is the verification complete ?

Yes,the verification is complete as there are no errors declared and it passes through all the tests.

