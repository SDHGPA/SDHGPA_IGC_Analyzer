# SDHGPA_IGC_Analyzer

Program which analyzes IGC files according to the rules of the San Diego Hang Gliding and Paragliding Association XC Contest.

## Description
The SDHGPA IGC Analyzer is a command line program that analyzes an IGC file.

In general, it looks for the maximum path distance for several kinds of paths.  These include straight-line, out-and-return, FAI triangle, and 4 segment paths.

It can optionally produce a KML file (for use in Google Earth) that will show the track and relevant turnpoints.

By default, it uses the WGS84 ellipsoid for distance calculations, but can optionally use the FAI sphere.

The straight-line, out-and-return, and FAI triangle calculations match the rules of the SDHGPA XC contest for hang gliders (https://www.sdhgpa.com/xc-contest---sdhgpa.html).  The 4-segment path conceptually matches what Leonardo (https://www.paraglidingforum.com/leonardo) calculates as the "Free Flight" distance (although Leonardo uses the FAI sphere, not the WGS84 ellipsoid).

## Dependencies
We depend on the GeographicLib library to do the WGS84 distance calculations.  See https://geographiclib.sourceforge.io/

## Issues
The program does not attempt to trim on-the-ground points, i.e., those where the pilot might be walking prior to launch, or after landing.  This should be a future enhancement.

The FAI now allows a triangle where the start point is not necessarily a vertex of the triangle.  The current calculation requires the start point to be a vertex of the triangle.  Assuming the SDGHPA adopts this change, this will be a future enhancement.

## Building the program
TBD.

## Running the program
The program is run like most command line tools.  Invoke the program, give it optional flags, then a list of IGC files to be analyzed.  The synopsis is:

```SDHGPA_IGC_Analyzer [-fwkdvh] [file ...]```

For example, `SDHGPA_IGC_Analyzer -h` will produce help output like this:

```
usage is
-f : Use FAI Sphere for distance calculations
-w : Use WGS84 for distance calculations (default if FAI Sphere is not specified)
-k : Produce KML file for each input file
-d : print debug info (verbose)
-v : print version
-h : help (usage)
```

The normal output of the program is text (and optionally, KML files).  For example, `SDHGPA_IGC_Analyzer 29IA0OV1.igc` will produce this:

```
29IA0OV1.igc
UTC Date: 09/18/22
PILOT:Jeff Brown
GPS Datum:WGS84
996 B records found
Launch  32,46.478   -116,28.604
Landing 32,45.818   -116,29.305

Analysis using WGS84 ellipsoid
Max from launch:  1.9618 km, 1.2190 mi
  A       32,46.478   -116,28.604
  F       32,47.423   -116,29.176
Straight Line:  3.9060 km, 2.4271 mi
  A       32,47.423   -116,29.176
  F       32,45.465   -116,28.235
Out & Return:   6.6941 km, 4.1595 mi, penalty 379.6 m
  A       32,47.238   -116,29.087
  B       32,45.465   -116,28.235
  F       32,47.026   -116,29.056
FAI Triangle:   4.4548 km, 2.7681 mi, penalty 0.0 m
  A       32,46.450   -116,28.680
  B       32,46.771   -116,29.394
  C       32,47.419   -116,29.145
  F       32,46.416   -116,28.749
  %s      28.3500%, 28.2672%, 43.3828%
4 segment:     12.3225 km, 7.6569 mi
  A       32,46.035   -116,28.452
  B       32,47.423   -116,29.176
  C       32,45.465   -116,28.235
  D       32,47.026   -116,29.056
  F       32,45.720   -116,29.345

HG XC Summary
PILOT:Jeff Brown
  max score = 4.991
    d1 = 4.160 mi OR UTC Date: 09/18/22
```
The first part is general data from the IGC file.  Latitude and longitude are displayed in degrees, decimal minutes format.  Positive corresponds with North and East, negative with South and West.  Distances are printed in kilometers and statute miles.

The second part has the various path calculations.  For each type of path, the total path distance is shown, followed by the lat/long of the specific points used in that path calculation.  `A` always represents the start point, and `F` always represents the finish (end) point.  `B`, `C`, `D` represent the first, second, and third turnpoint.

## Details
The "Max from launch" distance is simply the maximum distance from launch to any track point.

The "Straight Line" distance is simply the maximium distance between any two track points.

The "Out & Return" distance is the maximum two-segment path satisfying the following criteria.  The end point must be within 400 meters of the start point.  The distance calculated for the out and return flight will be the "out" distance (from start to turn point) plus the smaller of the "out" distance and the "return" distance (from turn point to end point).  If the "return" distance is less than the "out" distance, the difference is shown as "penalty" in the output.  Thus the O&R distance is twice the "out" distance minus the "penalty".

The "FAI Triangle" distance is the maximum three segment path satisfying the following criteria.  The end point must be within 400 meters of the start point.  The start point will be the first vertex of the triangle.  The shortest leg of the triangle must be at least 28% of the total triangular distance.  The distance given for the triangle flight will be the sum of the first leg, the second leg, and the smaller of the third leg and the distance from the 3rd vertex to the end point.  If the last segment is less than the last leg of the triangle, then the difference is shown as "penalty" in the output.  The output also shows the legs as a percentage of the total.

The "4 segment" distance is the maximum distance of a 4 segment path, that is, where there is a start point, three turnpoints, and an end point.

At the end, the program prints out a scoring summary intended for the hang gliding portion of the contest.  It assumes that all IGCs for all HG pilots for that year's contest are fed into the program.  Then it will summarize, for each pilot, their XC contest score and the flights that went into it.  The HG scoring takes the top two adjusted scores out of three categories, provided the scores are not from the same flight.  See the rules on the web site.  This section can be ignored for PG flights.
