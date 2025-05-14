# Compression 

## CompressSensor Tests

### 1. MaxValueTest

- **Purpose**: Tests delta encoding with max and min int32 values
- **Input**: values = [INT32_MAX, INT32_MAX, 0, INT32_MAX, INT32_MIN, INT32_MAX]
- **Expect**: compressedValues == [INT32_MAX, 0, -INT32_MAX, INT32_MAX, INT32_MIN-INT32_MAX, INT32_MAX-INT32_MIN]

### 2. EmptyIntValuesTest

- **Purpose**: Tests delta encoding with none values
- **Input**: values = []
- **Expect**: length(compressedValues) == 0

### 3. OneIntValuesTest

- **Purpose**: Tests delta encoding with constant positive values
- **Input**: values = [1]
- **Expect**: length(compressedValues) == 1
- **Expect**: compressedValues == [1]

### 4. SingleZeroIntValueTest

- **Purpose**: Tests delta encoding with one zero value
- **Input**: values = [0]
- **Expect**: length(compressedValues) == 1
- **Expect**: compressedValues == [0]

### 5. SingleNegativeIntValueTest

- **Purpose**: Tests delta encoding with one negative value
- **Input**: values = [-42]
- **Expect**: length(compressedValues) == 1
- **Expect**: compressedValues == [-42]

### 6. ConstPositiveIntValuesTest

- **Purpose**: Tests delta encoding with constant positive values
- **Input**: values = [1,1,1,1]
- **Expect**: length(compressedValues) == 4
- **Expect**: compressedValues == [1, 0, 0, 0]

### 7. ConstZeroIntValuesTest

- **Purpose**: Tests delta encoding with constant zero values
- **Input**: values = [0,0,0,0]
- **Expect**: length(compressedValues) == 4
- **Expect**: compressedValues == [0, 0, 0, 0]

### 8. ConstNegativIntValuesTest

- **Purpose**: Tests delta encoding with constant negative values
- **Input**: values = [-2,-2,-2,-2]
- **Expect**: length(compressedValues) == 4
- **Expect**: compressedValues == [-2, 0, 0, 0]

### 9. PositiveIntValuesPositiveDerivationTest

- **Purpose**: Tests delta encoding with positive and increasing values
- **Input**: values = [1, 2, 3, 4]
- **Expect**: length(compressedValues) == 4 
- **Expect**: compressedValues == [1, 1, 1, 1]

### 10. PositiveIntValuesNegativeDerivationTest

- **Purpose**: Tests delta encoding with positive and decreasing values
- **Input**: values = [10,9,8,7,6,1]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [10, -1, -1, -1, -1, -5]

### 11. NegativeIntValuesPositiveDerivationTest

- **Purpose**: Tests delta encoding with negative and increasing values
- **Input**: values = [-10,-9,-8,-7,-6,-1]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [-10,1,1,1,1,5]

### 12. NegativeIntValuesNegativeDerivationTest

- **Purpose**: Tests delta encoding with negative and decreasing values
- **Input**: values = [-10,-11,-13,-14,-18,-20]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [-10,-1,-2,-1,-4,-2]

### 13. MixedIntValuesNegativeDerivationTest

- **Purpose**: Tests delta encoding with mixed and decreasing values
- **Input**: values = [3,1,0,-3,-4,-6]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [3,-2,-1,-3,-1,-2]

### 14. MixedIntValuesPositiveDerivationTest

- **Purpose**: Tests delta encoding with mixed and increasing values
- **Input**: values = [-3,-1,0,1,1,3]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [-3,2,1,1,0,2]

### 15. PositiveIntValuesMixedDerivationTest

- **Purpose**: Tests delta encoding with positive and mixed derivation values
- **Input**: values = [1,2,3,2,1,0]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [1,1,1,-1,-1,-1]

### 16. MixedIntValuesmixedDerivationTest

- **Purpose**: Tests delta encoding with mixed and mixed derivation values
- **Input**: values = [-3,-2,1,0,1,-2]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [-3,1,3,-1,1,-3]

### 17. MixedIntValuesmixedDerivationTest2

- **Purpose**: Tests delta encoding with mixed and mixed derivation values
- **Input**: values = [3,2,-1,0,-1,2]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [3,-1,-3,1,-1,3]

### 18. NegativeIntValuesMixedDerivationTest

- **Purpose**: Tests delta encoding with negative and mixed derivation values
- **Input**: values = [-4, -3, -1, -1, -1, -2]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [-4,1,2,0,0,-1]

### 19. NegativeIntValuesMixedDerivationTest2

- **Purpose**: Tests delta encoding with negative and mixed derivation values
- **Input**: values = [-1, -5, -6, -4, -3, -2]
- **Expect**: length(compressedValues) == 6
- **Expect**: compressedValues == [-1,-4,-1,2,1,1]

### 20. PPGValuesTest

- **Purpose**: Tests delta encoding with values similar to a ppg signal
- **Input**: values = [1,2,4,7,8,9,7,6,7,6,5,4,3,2,1]
- **Expect**: length(compressedValues) == 15
- **Expect**: compressedValues == [1,1,2,3,1,1,-2,-1,1,-1,-1,-1,-1,-1,-1]

### 21. ACCValuesTest

- **Purpose**: Tests delta encoding with values similar to a acc signal
- **Input**: values = [1,1,1,2,1,1,300,1,1,1,2,1,-200,1,1]
- **Expect**: length(compressedValues) == 15
- **Expect**: compressedValues == [1,0,0,1,-1,0,299,-299,0,0,1,-1,-201,201,0]

### 21. ACCNormValuesTest

- **Purpose**: Tests delta encoding with values similar to a acc norm signal
- **Input**: values = [1.13,1.1234,1.2324,2.234,1.1243,1.3245,300.234,1.234,1.13,1.6578,2.456,1.2143,200.879,1.72389,1.124]
- **Expect**: length(compressedValues) == 15
- **Expect**: compressedValues == [1.13,1.1234,1.2324,2.234,1.1243,1.3245,300.234,1.234,1.13,1.6578,2.456,1.2143,200.879,1.72389,1.124]

### 22. ACCNormValuesTest

- **Purpose**: Tests delta encoding with values similar to a ecg signal
- **Input**: values = [0,2,3,2,0,0,-4,30,-6,1,1,3,4,3,1]
- **Expect**: length(compressedValues) == 15
- **Expect**: compressedValues == [0,2,1,-1,-2,0,-4,34,-36,7,0,2,1,-1,-2]