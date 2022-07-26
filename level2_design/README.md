# Bitmanipulation Coprocessor Design Verification

The verification environment is setup using [Vyoma's UpTickPro](https://vyomasystems.com) provided for the hackathon.

![Screenshot (139)](https://user-images.githubusercontent.com/105343698/182121300-ce03366f-226a-4ccf-bbd1-1714e3c7e9ff.png)


## Verification Environment

The [CoCoTb](https://www.cocotb.org/) based Python test is developed as explained. The test drives inputs to the Design Under Test (module mkbitmanip module here) which takes in three 32-bit inputs and 32-bit instruction.

The values are assigned to the input port using 
```

    # input transaction
    mav_putvalue_src1 = 0x5
    mav_putvalue_src2 = 0x0
    mav_putvalue_src3 = 0x0
    mav_putvalue_instr = 0x101010B3
```

The assert statement is used for comparing the mux's outut to the expected value.

The following error is seen:
```
    # comparison
    error_message = f'Value mismatch DUT = {hex(dut_output)} does not match MODEL = {hex(expected_mav_putvalue)}'
    assert dut_output == expected_mav_putvalue, error_message
```
## Test Scenario **(Important)**
- Test Inputs: 
```
    
    # input transaction
    mav_putvalue_src1 = 0x5
    mav_putvalue_src2 = 0x4
    mav_putvalue_src3 = 0x3
    mav_putvalue_instr =0x40007033
```
- Expected Output: EXPECTED OUTPUT=0x3
- Observed Output in the DUT:DUT OUTPUT=0x9

Output mismatches for the above inputs proving that there is a design bug

## Test Scenario **(Important)**
- Test Inputs: 
```
    # input transaction
    mav_putvalue_src1 = 0x34762354
    mav_putvalue_src2 = 0x364527
    mav_putvalue_src3 = 0x35638756
    mav_putvalue_instr =0x40007033
```
- Expected Output: EXPECTED OUTPUT=0x688044a1
- Observed Output in the DUT: DUT OUTPUT=0x6c0209

Output mismatches for the above inputs proving that there is a design bug

## Test Scenario **(Important)**
- Test Inputs: 
```
    # input transaction
    mav_putvalue_src1 = 0x4876
    mav_putvalue_src2 = 0x375f
    mav_putvalue_src3 = 0xabcd
    mav_putvalue_instr =0x40007033
```
- Expected Output: EXPECTED OUTPUT=0x9041
- Observed Output in the DUT: DUT OUTPUT=0xad

Output mismatches for the above inputs proving that there is a design bug

## Test Scenario **(Important)**
- Test Inputs: 
```
    # input transaction
    mav_putvalue_src1 = 0x0
    mav_putvalue_src2 = 0x375f
    mav_putvalue_src3 = 0xabcd
    mav_putvalue_instr =0x40007033
```
- Expected Output: EXPECTED OUTPUT=0x1
- Observed Output in the DUT: DUT OUTPUT=0x1

## Design Bug
Based on the above test input and analysing the design, we see the following observation 

```
    mav_putvalue_instr =0x40007033
```

When instruction value is ANDN(```0x40007033```), the design is containing a bug.
We can see that if ```mav_putvalue_src1``` is ```0x0``` , the test passes without failing otherwise it fails for all other values.



![Screenshot (143)](https://user-images.githubusercontent.com/105343698/182174340-51025531-6c77-4864-a343-da6175986e4b.png)


So Bitmanipulation Coprocessor design contains bug at  ```mav_putvalue_instr =0x40007033```


## Verification Strategy

The inputs are 32 bits which is quite long as each input has 4294967296 possible values and there are three inputs in the design. Parsing through all these combinations of inputs takes a long time and we have to consider the instruction inputs too.
A testbench which iterates through all the values possible is written in test_bit.py. Three loops parsing through the possible input values and instruction value will be running until the test case finds a error.

Other strategy is to give random inputs and check if the test case passes or fails.In this case, the manual work is more and we will not be able to know the exact number of errors and as the possible inputs are high.


## Is the verification complete ?

Giving random inputs produced one error where the bug is ```mav_putvalue_instr =0x40007033```

The inputs are quite long so the time to run the testbench is also high. The test_bit.py testbench is running too long and the terminal had to be killed, there were not many errors found through this method either.

