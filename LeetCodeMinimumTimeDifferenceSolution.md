## Approach #1: Brute Force [Time Limit Exceeded]
### Intuition and Algorithm

The brute force algorithm is to compute the difference between all timestamps, keeping track of the minimum along the way. However, we want to make sure that we aren't comparing a timestamp with itself: that is, do not compute the difference between timeStamp[i] and timeStamp[i] as that will lead to the wrong answer of 0.

To start with lets make a function that computes the difference between 2 timestamps. 
Comparing timestamps time1 and time2 is relatively easy if time2 > time1, but what if time1 > time2? A simple way around this is to increment time2 by 24 hours. Notice that difference("00:00", "23:59") can be more easily expressed as difference("24:00", "23:59"). To do that, we can make a function that increments a timestamp by 24 hours.
All it takes is some string splicing, and we get our code:

```java
/**
  * Returns value of time2 - time1 in minutes
  * If time2 < time1, addIncrement time2 by 24 hours
  */
private int difference(String time1, String time2) {
    if (time2.compareTo(time1) < 0) {
        time2 = add24(time2);
    }
    String[] t1 = time1.split(":");
    String[] t2 = time2.split(":");
    int minute1 = Integer.parseInt(t1[1]);
    int minute2 = Integer.parseInt(t2[1]);
    int hour1 = Integer.parseInt(t1[0]);
    int hour2 = Integer.parseInt(t2[0]);        
        
    return (hour2 * 60 + minute2) - (hour1 * 60 + minute1);
}

/**
  * Adds 24 hours to given string s
  */
  private String add24(String s) {
    String[] splice = s.split(":");
    int newHour = 24 + Integer.parseInt(splice[0]);
    
	return newHour + ":" + splice[1];
}
```

With those helper functions we can compute the difference between all N<sup>2</sup> combinations of our timestamp, here is the final solution in Java:

```java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        int min = Integer.MAX_VALUE;
        
        for (int i = 0; i < timePoints.size(); i++) {
            for (int j = 0; j < timePoints.size(); j++) {
                int diff = difference(timePoints.get(i), timePoints.get(j));
                
                // Make sure that i != j, otherwise we will get a wrong solution of 0
                if (i != j && diff >= 0) {
                    min = Math.min(min, diff);
                } 
            }
        }
        
        return min;
    }
    
    /**
      * Adds 24 hours to given string s
      */
    private String add24(String s) {
        String[] splice = s.split(":");
        int newHour = 24 + Integer.parseInt(splice[0]);
        
        return newHour + ":" + splice[1];
    }
    
    /**
      * Returns value of time2 - time1 in minutes
      * If time2 < time1, addIncrement time2 by 24 hours
      */
    private int difference(String time1, String time2) {
        if (time2.compareTo(time1) < 0) {
            time2 = add24(time2);
        }
        String[] t1 = time1.split(":");
        String[] t2 = time2.split(":");
        int minute1 = Integer.parseInt(t1[1]);
        int minute2 = Integer.parseInt(t2[1]);
        int hour1 = Integer.parseInt(t1[0]);
        int hour2 = Integer.parseInt(t2[0]);
        
        
        int diff = (hour2 * 60 + minute2) - (hour1 * 60 + minute1);
        return diff;
    }
}
```
Complexity Analysis

* Time Complexity is: 
> O(N^2), where N is the size of timeStamps. This is because we go through all timestamps for each timestamp.

* Space Complexity is:
> O(1), the only additional space we use is our min variable, which does not scale with the size of timeStamps.

---

## Approach #2: Sorting [Accepted]
### Intuition and Algorithm

For this solution, we first sort the timeStamps, then compare the difference between each adjacent timeStamp. Since the list will be sorted, each timeStamp only needs to be compared with the timeStamp adjacent to it (as opposed to being compared to all other timeStamps in our first approach). Just as before, we keep track of our min value as we do our comparisons. After comparing all adjacent timeStamps, we have to compare the first and the last timeStamps with each other. This is because they are technically adjacent as well. 

For example: ["00:00", "00:02", "23:59"], We mustn't forget to compare "23:59" to "00:00" since that leads to our true solution.

Note: Conveniently, using String comparison will suffice thanks to the format of our timeStamps, if the timeStamps were presented differently, we might need to make a custom Comparator

With that in mind and using the same helper functions as the brute force solution, we get a Java solution that looks like this:

```java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        // Sort points from earliest to latest
        Collections.sort(timePoints);
        
        int min = Integer.MAX_VALUE;
        
        // Iterate over sorted timestamps, update min appropriately
        for (int i = 0; i + 1 < timePoints.size(); i++) {
            String time1 = timePoints.get(i);
            String time2 = timePoints.get(i + 1);
            
            min = Math.min(min, difference(time1, time2));
        }
        
        // Check if the difference between latest and earliest time is smallest
        String time1 = timePoints.get(timePoints.size() - 1);
        String time2 = timePoints.get(0);
        
        return Math.min(min, difference(time1, time2));
    }
    
    /**
      * Adds 24 hours to given string s
      */
    private String add24(String s) {
        String[] splice = s.split(":");
        int newHour = 24 + Integer.parseInt(splice[0]);
        
        return newHour + ":" + splice[1];
    }
    
    /**
      * Returns value of time2 - time1 in minutes
      */
    private int difference(String time1, String time2) {
        if (time2.compareTo(time1) < 0) {
            time2 = add24(time2);
        }
        String[] t1 = time1.split(":");
        String[] t2 = time2.split(":");
        int minute1 = Integer.parseInt(t1[1]);
        int minute2 = Integer.parseInt(t2[1]);
        int hour1 = Integer.parseInt(t1[0]);
        int hour2 = Integer.parseInt(t2[0]);
        
        
        return (hour2 * 60 + minute2) - (hour1 * 60 + minute1);
    }
}
```

Complexity Analysis

Time Complexity is: 
O(N*log(N)), where N is the size of timeStamps. This is because of the sort. Afterwards we iterate over each timeStamp which costs N, but that gets dominated by the N*log(N) complexity

Space Complexity is:
O(1), the only additional space we use is our min variable, which does not scale with the size of timeStamps.

---
### Approach #3: Two Pass with Bucket [Accepted]
## Intuition and Algorithm

The problem with Approach #2 was that the sorting (N*log(N)) dominated our actual timeStamp traversal (N), if we can make the sorting complexity N, this will speed up our algorithm. Unfortunately, there is no known sorting that is faster than N*log(N)... or at least none that is comparison based :). For this solution, we take ideas from radix sort.

We first make an array of booleans of size 24 * 60 (number of minutes in a day). The contents of this bucket will indicate whether or not we have a timeStamp for a given minute of the day. As such, we need a helper method that converts a given timestamp to minutes after 00:00. Here's how that looks like in Java:

```java
/**
  * Converts given timestamp into minutes within the day
  */
private int convertToMinutes(String time) {
    String[] splice = time.split(":");
    int minute = Integer.parseInt(splice[1]);
    int hour = Integer.parseInt(splice[0]);
     
    return hour * 60 + minute;
}
```

In our first pass, we will iterate over the timestamp and convert them into minutes. That is:
Given minute = convertToMinutes(timeStamps[i]), bucket[minute] = true

However, note that if bucket[minute] is already true, this would mean that we have a duplicate timeStamp in our list, which means that we can immediately return 0.

Once we have our bucket, we will go for a second pass where we iterate through our bucket and find the smallest interval between 2 true values.
That is, suppose our list is:
```
["00:00", "00:03", "00:07"]
```
Then our bucket would be:
```
[T, F, F, T, F, F, F, T, ...]
```
Since the smallest interval between 2 true values is between index 3 and 0, our min value would be 3 - 0 = 3.

After this second pass, we must do what we did in Approach 2, and calculate the interval between the latest timestamp and the earliest timestamp + 24hrs.
Notice here that since we've converted our timeStamps into minutes, we would only need to increment our earliest timeStamp by 24 * 60 (24hrs converted to minutes).

After all our comparisons, we will get our minimum value.
With all that in mind, here is the solution in Java:

```java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        int minutesInDay = 24 * 60;
        boolean[] bucket = new boolean[minutesInDay];
        
        // Initialize array to see which timepoints are available to us
        for (String timePoint : timePoints) {
            int minute = convertToMinutes(timePoint);
            
            if (bucket[minute]) {
                return 0;
            } 
            
            bucket[minute] = true;
        }
        
        int min = Integer.MAX_VALUE;
        int previousTime = -1;
        int earliestTime = -1;
        
        // Iterate over all timestamps in "sorted" order and update min appopriately
        for (int i = 0; i < minutesInDay; i++) {
            if (bucket[i] && previousTime == -1) {
                previousTime = i;
                earliestTime = i;
            } else if (bucket[i]) {
                min = Math.min(min, i - previousTime);
                previousTime = i;
            }
        }
        
        // Checks if the difference between the latest date and earliest date is the minimum
        min = Math.min(min, minutesInDay + earliestTime - previousTime);
        
        return min;
    }
    
    /**
      * Converts given timestamp into minutes within the day
      */
    private int convertToMinutes(String time) {
        String[] splice = time.split(":");
        int minute = Integer.parseInt(splice[1]);
        int hour = Integer.parseInt(splice[0]);
        
        return hour * 60 + minute;
    }
}
```

Complexity Analysis

Time Complexity is: 
O(N), where N is the size of timeStamps. This is because we iterate over all our timeStamps in our first pass. Our second pass will do 24 * 60 = 1440 iterations regardless of N.

Space Complexity is:
O(1). Even though use our bucket as storage, its size will always be 24 * 60. Thus, our space is constant (unaffected by size of timeStamps)