# Decompression 

## DecompressSensor Tests

### 1. MaxValueTest

- **Purpose**: Tests delta encoding with max and min int32 values
- **Input**: compressedValues = [INT32_MAX, 0, -INT32_MAX, INT32_MAX, INT32_MIN-INT32_MAX, INT32_MAX-INT32_MIN]
- **Expect**: decompressedValues == [INT32_MAX, INT32_MAX, 0, INT32_MAX, INT32_MIN, INT32_MAX]

### 2. EmptyIntValuesTest

- **Purpose**: Tests delta encoding with none values
- **Input**: compressedValues = []
- **Expect**: length(decompressedValues) == 0

### 3. OneIntValuesTest

- **Purpose**: Tests delta encoding with constant positive values
- **Input**: compressedValues = [1]
- **Expect**: length(decompressedValues) == 1
- **Expect**: decompressedValues == [1]

### 4. SingleZeroIntValueTest

- **Purpose**: Tests delta encoding with one zero value
- **Input**: compressedValues = [0]
- **Expect**: length(decompressedValues) == 1
- **Expect**: decompressedValues == [0]

### 5. SingleNegativeIntValueTest

- **Purpose**: Tests delta encoding with one negative value
- **Input**: compressedValues = [-42]
- **Expect**: length(decompressedValues) == 1
- **Expect**: decompressedValues == [-42]

### 6. ConstPositiveIntValuesTest

- **Purpose**: Tests delta encoding with constant positive values
- **Input**: compressedValues = [1, 0, 0, 0]
- **Expect**: length(decompressedValues) == 4
- **Expect**: decompressedValues == [1,1,1,1]

### 7. ConstZeroIntValuesTest

- **Purpose**: Tests delta encoding with constant zero values
- **Input**: compressedValues = [0,0,0,0]
- **Expect**: length(decompressedValues) == 4
- **Expect**: decompressedValues == [0, 0, 0, 0]

### 8. ConstNegativIntValuesTest

- **Purpose**: Tests delta encoding with constant negative values
- **Input**: compressedValues = [-2, 0, 0, 0]
- **Expect**: length(decompressedValues) == 4
- **Expect**: decompressedValues == [-2,-2,-2,-2]

### 9. PositiveIntValuesPositiveDerivationTest

- **Purpose**: Tests delta encoding with positive and increasing values
- **Input**: compressedValues = [1, 1, 1, 1]
- **Expect**: length(decompressedValues) == 4 
- **Expect**: decompressedValues == [1, 2, 3, 4]

### 10. PositiveIntValuesNegativeDerivationTest

- **Purpose**: Tests delta encoding with positive and decreasing values
- **Input**: compressedValues = [10, -1, -1, -1, -1, -5]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [10,9,8,7,6,1]

### 11. NegativeIntValuesPositiveDerivationTest

- **Purpose**: Tests delta encoding with negative and increasing values
- **Input**: compressedValues = [-10,1,1,1,1,5]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [-10,-9,-8,-7,-6,-1]

### 12. NegativeIntValuesNegativeDerivationTest

- **Purpose**: Tests delta encoding with negative and decreasing values
- **Input**: compressedValues = [-10,-1,-2,-1,-4,-2]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [-10,-11,-13,-14,-18,-20]

### 13. MixedIntValuesNegativeDerivationTest

- **Purpose**: Tests delta encoding with mixed and decreasing values
- **Input**: compressedValues = [3,-2,-1,-3,-1,-2]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [3,1,0,-3,-4,-6] 

### 14. MixedIntValuesPositiveDerivationTest

- **Purpose**: Tests delta encoding with mixed and increasing values
- **Input**: compressedValues = [-3,2,1,1,0,2]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [-3,-1,0,1,1,3]

### 15. PositiveIntValuesMixedDerivationTest

- **Purpose**: Tests delta encoding with positive and mixed derivation values
- **Input**: compressedValues = [1,1,1,-1,-1,-1]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [1,2,3,2,1,0]

### 16. MixedIntValuesmixedDerivationTest

- **Purpose**: Tests delta encoding with mixed and mixed derivation values
- **Input**: compressedValues = [-3,1,3,-1,1,-3]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [-3,-2,1,0,1,-2]

### 17. MixedIntValuesmixedDerivationTest2

- **Purpose**: Tests delta encoding with mixed and mixed derivation values
- **Input**: compressedValues = [3,-1,-3,1,-1,3]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [3,2,-1,0,-1,2]

### 18. NegativeIntValuesMixedDerivationTest

- **Purpose**: Tests delta encoding with negative and mixed derivation values
- **Input**: compressedValues = [-4,1,2,0,0,-1]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [-4, -3, -1, -1, -1, -2]

### 19. NegativeIntValuesMixedDerivationTest2

- **Purpose**: Tests delta encoding with negative and mixed derivation values
- **Input**: compressedValues = [-1,-4,-1,2,1,1]
- **Expect**: length(decompressedValues) == 6
- **Expect**: decompressedValues == [-1, -5, -6, -4, -3, -2]

### 20. PPGValuesTest

- **Purpose**: Tests delta encoding with values similar to a ppg signal
- **Input**: compressedValues = [1,1,2,3,1,1,-2,-1,1,-1,-1,-1,-1,-1,-1]
- **Expect**: length(decompressedValues) == 15
- **Expect**: decompressedValues == [1,2,4,7,8,9,7,6,7,6,5,4,3,2,1]

### 21. ACCValuesTest

- **Purpose**: Tests delta encoding with values similar to a acc signal
- **Input**: compressedValues = [1,0,0,1,-1,0,299,-299,0,0,1,-1,-201,201,0]
- **Expect**: length(decompressedValues) == 15
- **Expect**: decompressedValues == [1,1,1,2,1,1,300,1,1,1,2,1,-200,1,1]

### 21. ACCNormValuesTest

- **Purpose**: Tests delta encoding with values similar to a acc norm signal
- **Input**: compressedValues = [1.13,1.1234,1.2324,2.234,1.1243,1.3245,300.234,1.234,1.13,1.6578,2.456,1.2143,200.879,1.72389,1.124]
- **Expect**: length(decompressedValues) == 15
- **Expect**: decompressedValues == [1.13,1.1234,1.2324,2.234,1.1243,1.3245,300.234,1.234,1.13,1.6578,2.456,1.2143,200.879,1.72389,1.124]

### 22. ACCNormValuesTest

- **Purpose**: Tests delta encoding with values similar to a ecg signal
- **Input**: compressedValues = [0,2,1,-1,-2,0,-4,34,-36,7,0,2,1,-1,-2]
- **Expect**: length(decompressedValues) == 15
- **Expect**: decompressedValues == [0,2,3,2,0,0,-4,30,-6,1,1,3,4,3,1]