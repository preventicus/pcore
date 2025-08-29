# pcore

## Documentation:
	
  This is a protobuf based file definition to save time based sensor data.

	The basic ideas in the pcore format are 

	 1) get rid of timestamps as good as possible 
	 2) to store differences instead of absolute values of sensor data.

	Currently, data from Photoplethysmograph (PPG), Accelerometer (ACC), 
	and Electrocardiogram (ECG) sensors are supported. Each sensor’s data 
	is stored in a uniform Sensor structure, which encapsulates 
	sensor-specific metadata and a sequence of measurement values. 
	Depending on the sensor type, these values are stored either as 
	integers (IntValues) or floating-point numbers (DoubleValues).

	Each sensor provides time-synchronized measurements across one or 
	multiple channels. The underlying timestamps for all channels of a 
	sensor are shared and compressed using a structure called CompressedTimestampsContainer. 
	This container represents the measurement timeline by dividing it into 
	sections of constant time intervals. The compressed representation stores 
	the initial timestamp, time deltas within a section, the time deltas between the 
	sections, and the number of data points per section.

	For data compression purposes, measurement values are stored as follow:
	 • Integer values are compressed as cumulative differences (deltas), starting from zero.
	 • Floating-point values are stored as-is, without compression.

	The compressed timestamp and values can later be reconstructed using the 
	provided decompression logic. This ensures both space-efficient storage and 
	accurate recovery of the original measurement series.

	The structure of the compressed format is defined in the Data message. 
	It includes metadata, the compressed timestamps, and a list of sensor messages. 
	For an overview of the involved data structures and their relationships, refer to 
	the class diagram below.

  We recommend naming the resulting data files *.pcore (PreventicusCore).

  The proto definition uses protobuf version 3. 
 
## Protobuf Struct

```mermaid
classDiagram

%% === Hauptstruktur ===

class Data {
  +metadata: Metadata
  %% oneof timestamps:
  +timestamps: CompressedTimestampsContainer
  +sensors: Sensor[]
}

%% === Metadata ===

class Metadata {
  +timezone_offset_min: sint32
  +pcore_version: Version
  +device: Device
}

class Device {
  +name: string
  +manufacturer: string
  +id: string
  +firmware_version: Version
}

%% === Timestamps ===

class CompressedTimestampsContainer {
  +first_unix_timestamp_ms: uint64
  +outer_sections_durations_ms: uint32[]
  +inner_sections_durations_ms: uint32[]
  +sections_sizes: uint32[]
}

%% === Sensor ===

class Sensor {
  %% oneof type:
  +type: Photoplethysmograph | Accelerometer | Electrocardiogram
  +values: IntValues | DoubleValues
}

class IntValues {
    values: sint64[]
}

class DoubleValues {
    values: double[]
}

%% === Accelerometer ===

class Accelerometer {
  +type: AccelerometerType
}

class AccelerometerType {
  <<enumeration>>
  TYPE_UNSPECIFIED
  TYPE_X_COORDINATE
  TYPE_Y_COORDINATE
  TYPE_Z_COORDINATE
  TYPE_EUCLIDEAN_DIFFERENCES_NORM
}

%% === Photoplethysmograph ===

class Photoplethysmograph {
  %% oneof light:
  +light: uint32 | PhotoplethysmographColor
}

class PhotoplethysmographColor {
  <<enumeration>>
  COLOR_UNSPECIFIED
  COLOR_GREEN
  COLOR_RED
  COLOR_BLUE
}

%% ====== Electrocardiogram ===

class Electrocardiogram {
    +channel: uint32
}

%% === Version ===

class Version {
  +major: uint32
  +minor: uint32
  +patch: uint32
}

%% === Beziehungen ===

Data --> Metadata
Metadata --> Device
Device --> Version : firmware_version
Metadata --> Version : pcore_version

Data --> CompressedTimestampsContainer

Data --> Sensor 
Sensor --> Photoplethysmograph
Sensor --> Accelerometer
Sensor --> Electrocardiogram
Sensor --> IntValues 
Sensor --> DoubleValues 

Photoplethysmograph --> PhotoplethysmographColor : color
Accelerometer --> AccelerometerType : type
```

## Compress

### Timestamp Compress

The basic idea behind compressing a sequence of unix timestamps is to group 
all timestamps that have a constant interval between them into sections.

For each section, only the first timestamp needs to be stored in full under 
first_unix_timestamp_ms. Since the time differences between consecutive 
timestamps within a section are equal by definition, this interval stored in
inner_sections_durations_ms only needs to be stored once per section.
If a section contains only a single timestamp, its inner section duration 
is defined to be 0. 

Additionally, the number of timestamps in each section is stored in sections_sizes.

To reconstruct the full sequence, it is also necessary to store the duration between 
the start of consecutive sections. This value—called the outer section duration—is 
defined as the time difference between the first timestamp of section N+1 and the 
first timestamp of section N. For the first section, the outer section duration is 
defined to be 0.

One possible approach to grouping timestamps into sections is to iterate through 
the array from left to right and check whether the time difference between consecutive 
timestamps remains constant. If the difference changes, a new section is started 
at that point.

The following algorithm demonstrates this approach:

**Algorithm: FindSectionIndices**
<pre>
Name: 
  FindSectionIndices

Input:
  unixTimestampsMs = [t₀, t₁, ..., tₙ]  // ordered list of Unix timestamps

Output:
  SectionIdxs = [s₀, s₁, ..., sₘ] // indices in the Timestamps array where new sections start

Procedure:
  If unixTimestampsMs is empty:
    Return

  Append 0 to SectionIdxs
  Set isNewSection ← TRUE

  For i from 1 to length(unixTimestampsMs) - 1:
    Set duration ← unixTimestampsMs[i] - unixTimestampsMs[i - 1]

    If isNewSection:
      Set referenceDuration ← duration
      Set isNewSection ← FALSE

    If duration ≠ referenceDuration:
      Append i to SectionIdxs
      Set isNewSection ← TRUE

  Return SectionIdxs
</pre>

<pre>
Name: 
  CompressTimestamp

Input:
  unixTimestampsMs

Output:
  firstUnixTimestampMs
  innerSectionsDurationsMs
  outerSectionsDurationsMs
  sectionsSizes

Procedure:
  Set sectionIdxs ← FindSectionIndices(unixTimestampsMs)
  Set numberOfSections ← length(sectionIdxs)

  If numberOfSections == 0
    Return

  Set sizeUnixTimestamps ← length(unixTimestampsMs)
  Set firstUnixTimestampMs ← unixTimestampsMs[0]
  Append 0 to outerSectionsDurationsMs

  If numberOfSections == 1
    If sizeUnixTimestamps > 1
	  Append unixTimestampsMs[1] - firstUnixTimestampMs to innerSectionsDurationsMs
    Else
	  Append 0 to innerSectionsDurationsMs    
    Append sizeUnixTimestamps to sectionsSizes
    Return

  Append unixTimestampsMs[1] - firstUnixTimestampMs to innerSectionsDurationsMs
  Append sectionIdxs[1] to sectionsSizes

  For i from 1 to numberOfSections - 2
    Set previousSectionIndex ← sectionIdxs[i - 1]
    Set currentSectionIndex ← sectionIdxs[i]
    Set nextSectionIndex ← sectionIdxs[i + 1]
    Append unixTimestampsMs[currentSectionIndex + 1] - unixTimestampsMs[currentSectionIndex] to innerSectionsDurationsMs
    Append unixTimestampsMs[currentSectionIndex] - unixTimestampsMs[previousSectionIndex] to outerSectionsDurationsMs
    Append nextSectionIndex - currentSectionIndex to sectionsSizes

  Set lastSectionIndex ← sectionIdxs[numberOfSections - 1]
  Append unixTimestampsMs[lastSectionIndex] - unixTimestampsMs[sectionIdxs[numberOfSections - 2]] to outerSectionsDurationsMs
  If sizeUnixTimestamps - 1 == lastSectionIndex:
    Append 0 to innerSectionsDurationsMs 
  Else
    Append unixTimestampsMs[lastSectionIndex + 1] - unixTimestampsMs[lastSectionIndex] to innerSectionsDurationsMs
  Append sizeUnixTimestamps - lastSectionIndex to sectionsSizes
</pre>

### Compress Integer Value

To compress integer values in Protobuf, it is generally more efficient to 
store smaller numbers. Therefore, the basic idea is to save the differences 
between consecutive sensor data values. This approach is particularly effective 
for sensor data that changes slowly over time. For the first value in the sequence, 
the difference is defined as the difference between that value and zero.

<pre>
Name:
  CompressIntergerValues

Input:
  values // list of integers (IntValues.values)

Output:
  compressValues

Procedure:
  If values is empty:
    Return

  Append values[0] to compressValues
  For i from 1 to length(values) - 1:
    Append values[i] - values[i - 1] to compressValues
</pre>

### Compress Double Value

Protobuf cannot further compress double-precision floating-point values. 
Therefore, double values are stored as-is.

<pre>
Name:
  CompressDoupleValues

Input:
  values // list of doubles (DoubleValues.values)

Output:
  compressedValues

Procedure:
  If values is empty:
    Return

  For each value in values:
    Append value to compressedValues
</pre>

## Decompress

### Decompress Compressed Timestamp
<pre>
Name:
  DecompressTimestamps

Input:
  firstUnixTimestampMs
  outerSectionsDurationsMs
  innerSectionsDurationsMs
  sectionsSizes

Output:
  unixTimestampsMs

Procedure:
  Set currentTimestampMs ← firstUnixTimestampMs
  For i from 0 to length(sectionsSizes) - 1:
    Set currentTimestampMs ← currentTimestampMs + outerSectionsDurationsMs[i]
    For j from 0 to sectionsSizes[i] - 1:
      Append currentTimestampMs + j * innerSectionsDurationsMs[i] to unixTimestampsMs
</pre>

### Decompress Integer Value
<pre>
Name:
  DecompressIntegerValue

Input:
  compressedValues // list of differences (IntValues)

Output:
  values // list of integers

Procedure:
  If compressedValues is empty
    Return []

  Set cumulativeSum ← 0
  For each difference in compressedValues
	Set cumulativeSum ← cumulativeSum + difference
    Append cumulativeSum to values

  Return values
</pre>

### Decompress Double Value
<pre>
Name:
  DecompressDoubleValue

Input:
  compressedValues // list of doubles (DoubleValues)

Output:
  values // list of doubles

Procedure:
  For each value in compressedValues
    Append value to values
</pre>

## Validation 

### Compress

Data is only allowed to be compressed if:

1. there are also Unixtimestamps for the sensor data 
2. and each sensor has the same number of elements as there are unixtimestamps. 
3. time_zone_offset is between -720 and 840

### Decompress

Data is only allowed to be decompressed if:

1. there are also Unixtimestamps for the sensor data
2. and each sensor has the same number of elements
3. the sum of sections_sizes is equal to the number of sensor elements.
4. the first element of outer_sections_durations_ms must allways be 0.
5. the remaining elements of outer_sections_durations_ms must allways greater than 0.
6. all but the last element of inner_sections_durations_ms must always be greater than 0. The last element may also be 0.
7. if the sum of sections_sizes is equal to 1 the only element of inner_sections_durations_ms must be 0.
8. all elements of sections_sizes must allways be greater than 0.
9. time_zone_offset is between -720 and 840.

## Example

    (each | marks a timestamp, the difference between two | with no space is for this 
    example defined 40 milliseconds, so 1 space is 80 milliseconds)

      Sequence 1   Sequence 2   Sequence 3   Sequence 4
      ||||||||||   ||||||||     | | | | |      |||   
    ──↑────────────↑────────────↑─↑────────────────────>        
      a            b            c d                 time

      a: first_unix_timestamp_ms (begin measurement)
      c to d: inner_sections_durations_ms (time difference in each section)
      a to a: outer_sections_durations_ms[0] (allways 0)
      a to b: outer_sections_durations_ms[1] (time differences to predecessor start of section)
      b to c: outer_sections_durations_ms[2] (and so on)


    unix,         ppg
    1675732789987,38763 <- Begin Section 1
    1675732790027,38771
    1675732790067,38780
    1675732790107,38793
    1675732790147,38784
    1675732790187,38780
    1675732790227,38780
    1675732790267,38783
    1675732790307,38790
    1675732790347,38782
    1675732790467,46321 <- Begin Section 2          
    1675732790507,46327
    1675732790547,46318
    1675732790587,46316
    1675732790627,46313
    1675732790667,46313
    1675732790707,46313
    1675732790747,46336
    1675732790867,58772 <- Begin Section 3
    1675732790947,58774
    1675732791027,58775
    1675732791107,58776
    1675732791187,58773
    1675732791347,19982 <- Begin Section 4
    1675732791387,19982
    1675732791427,19978

    leads to:

    Data: [{
      CompressedTimestampsContainer: {
        first_unix_timestamp_ms: 1675732789987,
        outer_sections_durations_ms: [0, 480, 400, 480],
        inner_sections_durations_ms: [40, 40, 80, 40],
        sections_sizes: [10, 8, 5, 3]
      },
      Sensors: [{
        IntValuesContainer: [{
          values: [38763, 8, 9, 13, -9, -4, 0, 3, 7, -8, 7539, 6, -9, -2, -3, 0, 0, 23, 12436, 2, 1, 1, -3, -38791, 0, -4]
        }]
      }]
    }]