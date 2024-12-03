# Run Analitics: Extracting and Analyzing Data from Fitness Trackers TCX files

Parse the tcx (xml file) from a sportwatch (Garmin,Strava,GoogleFit,MapMyRide etc.), create dictionary with xml data and plot everything .

*[View on GItHub](https://github.com/XarisA/runalysis)*

## Sample Plots

![Heart Rate per Distance](https://user-images.githubusercontent.com/3985557/165562301-cd274c3d-1e9f-4a6b-9f23-7b78494fb8a2.png)

![Heart Rate Zones %](https://user-images.githubusercontent.com/3985557/165562310-41c43e56-8d38-4018-94c6-3630c15f6c3d.png)

## Parsed Data

```python
MyDictionary= ttd().dict_data

print MyDictionary
```

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

_'2017-02-11T06:48:34.000Z'_

### _Get Distance_

```python
MyDictionary[0,1][1]
```

_3.760009765625_

### _Get Heart Rate _

```python
MyDictionary[0,1][2]
```
_87_

