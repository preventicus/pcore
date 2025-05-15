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

- **Purpose**: Tests delta encoding with real ppg values
- **Input**: compressedValues = [1405,-26,46,103,20,33,49,40,32,15,124,129,106,-12,171,-6,91,80,147,161,109,146,-43,-357,-552,-336,-68,19,75,111,31,-38,89,89,32,106,176,77,63,35,150,118,125,354,112,136,132,134,87,299,94,318,94,247,242,161,134,335,225,93,249,364,76,446,269,335,189,313,220,262,461,224,343,-546,-1962,-2206,-1151,-801,-420,-27]
- **Expect**: length(decompressedValues) == 80
- **Expect**: decompressedValues == [1405,1379,1425,1528,1548,1581,1630,1670,1702,1717,1841,1970,2076,2064,2235,2229,2320,2400,2547,2708,2817,2963,2920,2563,2011,1675,1607,1626,1701,1812,1843,1805,1894,1983,2015,2121,2297,2374,2437,2472,2622,2740,2865,3219,3331,3467,3599,3733,3820,4119,4213,4531,4625,4872,5114,5275,5409,5744,5969,6062,6311,6675,6751,7197,7466,7801,7990,8303,8523,8785,9246,9470,9813,9267,7305,5099,3948,3147,2727,2700]

### 21. ACCValuesTest

- **Purpose**: Tests delta encoding with real acc values
- **Input**: compressedValues = [-909,1,-7,-1,3,0,7,10,5,5,1,1,4,-3,-1,-2,-2,-4,-3,-1,-11,-5,-11,7,-3,-3,12,6,6,9,6,9,17,8,6,6,-9,-19,-15,-8,-9,-2,-9,0,-2,-7,-7,-4,-6,-6,-6,-4,-6,-3,-4,-4,5,2,8,7,8,9,3,3,-6,-7,-7,-8,-7,-14,-10,-8,-2,-1,-2,-4,-5,-13,-15,-3,-1,-2,-1,-1,0,4,-12,-11,-25,-17,-20,-14,-20,-23,-35,-40,-77,-83,-94,-78,-81,-75,-47,-29,-9,7,14,24,26,40,22,42,27,26,38,20,13,0,-4,-3,4,8,10,11,19,25,23,36,25,40,28,37,20,14,22,5,-15,-33,-60,8,75,152,144,118,65,76,85,97,78,34,9,17,40,50,34,50,55,41,37,39,21,35,40,29,24,40,21,6,6,-11,-5,-9,-17,-21,-21,-9,-5,0,0,3,7,5,-1,-2,1,2,-5,-8,-21,-50,-38,-39,-25,-36,-29,-62,-70,-75,-50,-10,-6,-4,-18,-11,-24,-20,-23,-24,-23,-12,-18,-33,-14,-1,1,0,20,33,8,-30,-58,-77,-55,-78,-150,-231,-305,-417,-555,-720,-1116,-1223,64,1051,1004,750,381,55,47,123,55,46,81,171,193,185,193,176,141,89,40,-9,-26,-47,-46,-48,-51,-35,-26,-18,-11,9,-9,-13,-36,-32,-48,-39,-22,0,10,16,11,-1,-19,-12,13,34,67,96,104,126,112,88,54,25,6,8,19,46,69,80,50,42,21,-18,-37,-48,-91,-98,-85,-87,-76,-44,-27,-37,-54,-69,-83,-106,-105,-67,-62,-15,13,12,44,45,10,0,8,4,7,31,37,52,57,49,36,56,43,52,37,20,3,4,-4,-1,1,6,4,0,-1,-7,-10,-29,-24,-40,-23,-19,-16,-11,-4,-4,6,3,8,-4,2,-4,2,-8,0,-10,-1,0,4,3,7,5,0,1,5,-2,5,0,10,-7,-2,-1,5,3,1,0,-9,-9,3,-13,2,2,3,13,11,13,5,2,1,3,-2,-3,-1,-2,-2,-7,-11,-15,-15,-7,-4,-7,-5,-4,-2,-1,1,7,2]
- **Expect**: length(decompressedValues) == 417
- **Expect**: decompressedValues == [-909,-908,-915,-916,-913,-913,-906,-896,-891,-886,-885,-884,-880,-883,-884,-886,-888,-892,-895,-896,-907,-912,-923,-916,-919,-922,-910,-904,-898,-889,-883,-874,-857,-849,-843,-837,-846,-865,-880,-888,-897,-899,-908,-908,-910,-917,-924,-928,-934,-940,-946,-950,-956,-959,-963,-967,-962,-960,-952,-945,-937,-928,-925,-922,-928,-935,-942,-950,-957,-971,-981,-989,-991,-992,-994,-998,-1003,-1016,-1031,-1034,-1035,-1037,-1038,-1039,-1039,-1035,-1047,-1058,-1083,-1100,-1120,-1134,-1154,-1177,-1212,-1252,-1329,-1412,-1506,-1584,-1665,-1740,-1787,-1816,-1825,-1818,-1804,-1780,-1754,-1714,-1692,-1650,-1623,-1597,-1559,-1539,-1526,-1526,-1530,-1533,-1529,-1521,-1511,-1500,-1481,-1456,-1433,-1397,-1372,-1332,-1304,-1267,-1247,-1233,-1211,-1206,-1221,-1254,-1314,-1306,-1231,-1079,-935,-817,-752,-676,-591,-494,-416,-382,-373,-356,-316,-266,-232,-182,-127,-86,-49,-10,11,46,86,115,139,179,200,206,212,201,196,187,170,149,128,119,114,114,114,117,124,129,128,126,127,129,124,116,95,45,7,-32,-57,-93,-122,-184,-254,-329,-379,-389,-395,-399,-417,-428,-452,-472,-495,-519,-542,-554,-572,-605,-619,-620,-619,-619,-599,-566,-558,-588,-646,-723,-778,-856,-1006,-1237,-1542,-1959,-2514,-3234,-4350,-5573,-5509,-4458,-3454,-2704,-2323,-2268,-2221,-2098,-2043,-1997,-1916,-1745,-1552,-1367,-1174,-998,-857,-768,-728,-737,-763,-810,-856,-904,-955,-990,-1016,-1034,-1045,-1036,-1045,-1058,-1094,-1126,-1174,-1213,-1235,-1235,-1225,-1209,-1198,-1199,-1218,-1230,-1217,-1183,-1116,-1020,-916,-790,-678,-590,-536,-511,-505,-497,-478,-432,-363,-283,-233,-191,-170,-188,-225,-273,-364,-462,-547,-634,-710,-754,-781,-818,-872,-941,-1024,-1130,-1235,-1302,-1364,-1379,-1366,-1354,-1310,-1265,-1255,-1255,-1247,-1243,-1236,-1205,-1168,-1116,-1059,-1010,-974,-918,-875,-823,-786,-766,-763,-759,-763,-764,-763,-757,-753,-753,-754,-761,-771,-800,-824,-864,-887,-906,-922,-933,-937,-941,-935,-932,-924,-928,-926,-930,-928,-936,-936,-946,-947,-947,-943,-940,-933,-928,-928,-927,-922,-924,-919,-919,-909,-916,-918,-919,-914,-911,-910,-910,-919,-928,-925,-938,-936,-934,-931,-918,-907,-894,-889,-887,-886,-883,-885,-888,-889,-891,-893,-900,-911,-926,-941,-948,-952,-959,-964,-968,-970,-971,-970,-963,-961]

### 22. ACCNormValuesTest

- **Purpose**: Tests delta encoding with real acc norm values
- **Input**: compressedValues = [975.9780,983.0051,976.6443,976.3959,976.8055,976.3140,973.7654,974.8728,976.9058,971.5750,971.6965,977.5822,975.3389,977.1433,980.6783,981.7836,976.9058,980.5085,981.2003,985.6440,982.6067,981.4199,977.6144,978.7206,977.8645,976.7236,979.0056,978.7589,975.0287,973.9225,973.0468,972.3893,977.0005,971.1658,973.1531,974.1401,972.6567,971.3460,972.5071,972.0566,973.9168,977.6896,978.6782,972.0561,973.9790,976.6074,977.8231,976.7139,974.6143,973.0267,975.9990,971.8863,973.1444,972.1250,970.9501,971.8863,974.0996,972.4263,972.6145,980.3515,974.9103,971.9408,973.2518,974.2212,981.0973,976.7236,978.9408,975.7792,974.8066,977.1346,975.2748,977.3198,975.7095,975.3958,973.1721,975.1533,975.5209,974.0457,976.3473,984.8705]
- **Expect**: length(decompressedValues) == 80
- **Expect**: decompressedValues == [975.9780,983.0051,976.6443,976.3959,976.8055,976.3140,973.7654,974.8728,976.9058,971.5750,971.6965,977.5822,975.3389,977.1433,980.6783,981.7836,976.9058,980.5085,981.2003,985.6440,982.6067,981.4199,977.6144,978.7206,977.8645,976.7236,979.0056,978.7589,975.0287,973.9225,973.0468,972.3893,977.0005,971.1658,973.1531,974.1401,972.6567,971.3460,972.5071,972.0566,973.9168,977.6896,978.6782,972.0561,973.9790,976.6074,977.8231,976.7139,974.6143,973.0267,975.9990,971.8863,973.1444,972.1250,970.9501,971.8863,974.0996,972.4263,972.6145,980.3515,974.9103,971.9408,973.2518,974.2212,981.0973,976.7236,978.9408,975.7792,974.8066,977.1346,975.2748,977.3198,975.7095,975.3958,973.1721,975.1533,975.5209,974.0457,976.3473,984.8705]

### 23. ECGValuesTest

- **Purpose**: Tests delta encoding with real ecg values
- **Input**: compressedValues = [87,-37,-71,35,36,-83,-5,122,-7,-135,8,66,29,80,-154,-348,-108,41,121,41,-191,-157,-312,-435,275,827,289,-171,-41,-7,-138,-53,312,183,-379,-440,-428,-481,3,423,870,1372,362,-940,-454,280,-46,-286,134,157,-205,-154,-3,-14,31,115,75,-112,-312,22,460,-12,-354,-2,48,12,195,111,-24,-27,-320,-358]
- **Expect**: length(decompressedValues) == 72
- **Expect**: decompressedValues == [87,50,-21,14,50,-33,-38,84,77,-58,-50,16,45,125,-29,-377,-485,-444,-323,-282,-473,-630,-942,-1377,-1102,-275,14,-157,-198,-205,-343,-396,-84,99,-280,-720,-1148,-1629,-1626,-1203,-333,1039,1401,461,7,287,241,-45,89,246,41,-113,-116,-130,-99,16,91,-21,-333,-311,149,137,-217,-219,-171,-159,36,147,123,96,-224,-582]