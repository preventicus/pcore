# pcore


	Documentation:
	==============

    We recommend naming the resulting data files *.pcore (PreventicusCore).

	The basic ideas in the pcore format are 

		1) get rid of timestamps as good as possible 
		2) to store differences instead of absolute values of sensor data.

	Currently PPG sensors and ACC sensors are supported. The PPG data is stored in the PpgSensor message. The ACC data in the
	AccSensor message. The difference between the AccSensor and the PpgSensor is only in some meta information. The respective
	raw data are stored in the same way. Channels and Blocks are defined for this purpose. Each sensor has several channels. 
	The difference between the sensor and the channels is, that each data stream in a channel is based in the same timestamps. 
	A block is defined as a summary of data with equal time differnces. So, a block is completed when the time difference 
	between 2 timestamps is not equal to the time difference of the timestamps before. The pseudocode findBlocks describes how 
	the blocks can be found. See also th example below In each sensor the message timestamp is stored.  The start of the 
	measurement with this sensor is stored in unix_0 as unix timestamp in milliseconds. For each block the existing time 
	difference is stored in this block in d_unix. Also for each block the time differnce to its predecessor is saved in 
	d_unix_0. The pseudocode cutUnixInBlocks describes how the time data can be calculated. For each block, the data values 
	are stored as differences. Where the first data value is defined as a difference to 0. See the pseudocode 
	cutValuesInBlocks and createValueBlock. 

	Pseudocode:
	-----------

	findBlocks
	 * in: unix is an array of timestamps
	 * out: array of indices

        set referenceTimeDifference to 0
        set isNewBlock to true 
        create blocksIdxs as empty array

		add 0 to blocksIdxs

		loop i from 1 to lenght of unix 
			if isNewBlock
				set referenceTimeDifference to unix[i] - unix[i-1]
				set isNewBlock to false

			if unix[i] - unix[i-1] != referenceTimeDifference
				add i to blocksIdxs
				set isNewBlock to true

        return blocksIdxs

	cutUnixInBlocks
	 * in: unix is an array of timestamps,
	       blocksIdx is an array of indices describing the begin of a block
	 * out: dUnix
	        dUnix0

        create dUnix as empty array
        create dUnix0 as empty array

		add 0 to dUnix0
		add unix[1]-unix[0] to dUnix

        loop i from 1 to length of blocksIdx 
			set prevIdx to blocksIdx[i-1]
			set idx to blocksIdx[i]

			add unix[idx+1] - unix[idx] to dUnix
			add unix[idx] - unix[prevIdx] to dUnix0

        return dUnix, dUnix0
        
	cutValuesInBlocks
	 * in: values is the array of the data values i.e. ppg
	       blocksIdx is an array of indices describing the begin of a block
	 * out: array of blocks

	 	set n to length of blocksIdx
		create blocks as empty array

		loop i from 0 to n - 1
			add result of createValueBlock(blocksIdx[i], blocksIdx[i+1] - 1, values) to blocks
		
		set m to length of values
		add result of createValueBlock(blocksIdx[n-1], m - 1, values) to blocks

        return blocks

    createValueBlock
	 * in: fromIdx is the indice of the start of the block,
	       toIdx is the indice of the end of the block,
		   values is the array of the data values i.e. ppg
	 * out: the block structure 

	 	create dValue as empty array

		add values[fromIdx] to dValue // difference to 0
        
		loop i from fromIdx + 1 to toIdx 
			add values[i] - values[i-1] to dValue

		create block as block structure
		add dValue to block 

        return block

	Example:
	--------

	   Block 1     Block 2   Block 3   Block 4
	  ||||||||||  ||||||||  | | | | |   |||             (each | marks a timestamp, the difference between two | with no space 
	──↑───────────↑─────────↑─↑─────────────────>        is for this example defined 40 milliseconds, so 1 space is 80 milliseconds)
	  a           b         c d                time

	  a: unix_0 (begin measurement)
	  c to d: d_unix (time difference in each block)
	  a to a: d_unix_0[0] (allways 0)
	  a to b: d_unix_0[1] (time differences to predecessor start of block )
	  b to c: d_unix_0[2] (and so on)


	unix,         ppg
	1675732789987,38763 <- Begin Block 1
	1675732790027,38771
	1675732790067,38780
	1675732790107,38793
	1675732790147,38784
	1675732790187,38780
	1675732790227,38780
	1675732790267,38783
	1675732790307,38790
	1675732790347,38782
	1675732790467,46321 <- Begin Block 2          
	1675732790507,46327
	1675732790547,46318
	1675732790587,46316
	1675732790627,46313
	1675732790667,46313
	1675732790707,46313
	1675732790747,46336
	1675732790867,58772 <- Begin Block 3
	1675732790947,58774
	1675732791027,58775
	1675732791107,58776
	1675732791187,58773
	1675732791347,19982 <- Begin Block 4
	1675732791387,19982
	1675732791427,19978

	leads to:

	ppgSensors: [{
		timestamp {
			unix_0: 1675732789987
			d_unix_0: [0, 480, 400, 480]
			d_unix: [40, 40, 80, 40]
		}
		channels: [{
			blocks: [{
				d_value: [38763, 8, 9, 13, -9, -4, 0, 3, 7, -8]
			},
			{
				d_value: [46321, 6, -9, -2, -3, 0, 0, 23]
			},
			{
				d_value: [58772, 2, 1, 1, -3]
			},
			{
				d_value: [19982, 0, -4]
			}]
		}]
	}]

