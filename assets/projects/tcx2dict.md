# tcx2dict: A Python Module to parce TCX file to a Dictionary

***tcx2dict*** is a python module that reads data from a TCX file and stores them in a dictionary having a tuple as a key  
`(#Lap,#measurement)`.  
You can export your data from Garmin,Strava or any alternative and get them in python.
Use this dictionary to analyse the data by yourself.

*[View on GitHub](https://github.com/XarisA/tcx2dict)*


### Example:

```python
MyDictionary= ttd().dict_data
print MyDictionary
```

`Running: /home/xaris/bin/git/tcx2dict/tcx2dict/ttd.py (Sat Feb 18 17:40:10 2017)`

```xml
{(0, 1): ['2017-02-11T06:48:34.000Z', 3.760009765625, 87], (3, 35): ['2017-02-11T07:03:19.000Z', 3395.360107421875, 160], (4, 36): ['2017-02-11T07:07:46.000Z', 4291.8798828125, 167], (0, 76): ['2017-02-11T06:48:16.000Z', 382.6600036621094, 149], (4, 66): ['2017-02-11T07:08:54.000Z', 4519.06005859375, 168], (1, 64): ['2017-02-11T06:55:04.000Z', 1740.800048828125, 168], (2, 78): ['2017-02-11T07:00:54.000Z', 2913.080078125, 170], (0, 98): ['2017-02-11T06:49:02.000Z', 532.0800170898438, 147], (3, 86): ['2017-02-11T07:06:20.000Z', 4001.5400390625, 171]...}
```

## Data Explanation:

```python
MyDictionary[0,1]
```
_returns data from 1st Lap , 2nd Measurement._

`['2017-02-11T06:48:34.000Z', 3.760009765625, 87]`


### _Get Datetime in isoformat_

```python
MyDictionary[0,1][0]
```

`2017-02-11T06:48:34.000Z`

### _Get Distance_

```python
MyDictionary[0,1][1]
```

`3.760009765625`

### _Get Heart Rate_

```python
MyDictionary[0,1][2]
```
`87`
