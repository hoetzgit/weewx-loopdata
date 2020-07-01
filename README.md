# weewx-loopdata
*Open source plugin for WeeWX software.

Copyright (C)2020 by John A Kline (john@johnkline.com)

**This extension requires Python 3 and WeeWX 4.**

## Description

LoopData is a WeeWX service that generates a json file (loop-data.txt)
containing values for the observations in the loop packet; along with
today's high, low, sum, average and weighted averages for each observation
in the packet.

Another WeeWx report is specified in the LoopData configuration (e.g.,
`SeasonsReport`).  With this information, LoopData automatically converts
all values to the units called for in the report and also formats all
readings according to the report specified.  Thus, it is simple to replace
the reports observations in javascript as they will already be in the
correct units and in the correct format.  Formatted values are requested
by prefixing the name with 'FMT_'.  For example:
  * windSpeed:        14.1
  * FMT_windSpeed     14.1 mph
  * AVG_windSpeed     7.6
  * FMT_AVG_windSpeed 7.6 mph

Typically, loop packets are generated frequently in WeeWX
(e.g, every 2s).  With the constantly updating loop-data.txt file, one
can write javascript to update a report's html page in near real time.

LoopData adds all observations seen in loop packets.  Since loop data examines
loop packets to decide what data to write, it is easily extensible.  For
example, if the [weewx-purple](https://github.com/chaunceygardiner/weewx-purple)
extension is installed, the following observations immedately become available
in loopdata: `pm1_0`, `pm2_5`, `pm10_0` and `pm2_5_aqi`.  If you add another
sensor to your weather stattion and that sensor starts showing up in loop
packets, it will also become available in LoopData.

Lastly, in addtion to the fields generated by virtue of being in loop packets,
LoopData also makes avaiable 10 minute high wind gust, 24 hour barometer trend
and windrose data.

LoopData is easy to set up.  Specify the report to target and a json file will
be generated on every loop with observations, hights, lows, sums, averages and
weighted averages will formatted and converted for that report.  That is, the
units and format for observations areappropriate for the specified the report.

From there, just write javascript to load the json file on a regular basis
(typically, at the same rate as loop packets are generated in WeeWX) and update
your report's html page.

### Why?  What does it do?

LoopData writes a json file for every loop packet generated by WeeWX.  Just add
some javascript to a skin to take advantage of the json file to update the
report in near real time.

See weewx-loopdata in action with a WeatherBoard&trade; skin at
[www.paloaltoweather.com/weatherboard/](https://www.paloaltoweather.com/weatherboard/)
and in a "LiveSeasons" skin at
[www.paloaltoweather.com/](https://www.paloaltoweather.com/).

A WeatherBoard&trade; screenshot is below.

![Weatherbaord&trade; Report](WeatherBoard.png)

This extension was inspired by [weewx-realtime_gauge_data](https://github.com/gjr80/weewx-realtime_gauge-data).
This does not attempt to duplicate Gary's fantastic realtime gauge data plugin
for SteelSeries gauges.  In fact, I use that great extension [here](https://www.paloaltoweather.com/LiveSeasonsGauges/).
To power Steel Series gauges from WeeWX, you definitely want to use Gary's extension.

# Installation Instructions
1. cd to the directory where you have cloned this extension, e.g.,

   `cd ~/software/weewx-loopdata`
1. Run the following command.

   `sudo /home/weewx/bin/wee_extension --install .`

   Note: this command assumes weewx is installed in /home/weewx.  If it's installed
   elsewhere, adjust the path of wee_extension accordingly.

1. The install creates a LoopData section in weewx.conf as shown below.  Adjust
   the values accordingly.  In particular, specify the `target_report` for the
   report you wish to use for formatting and units.

```
[LoopData]
    [[FileSpec]]
        loop_data_dir = /home/weewx/loop-data
        filename = loop-data.txt
    [[Formatting]]
        target_report = SeasonsReport
    [[RsyncSpec]]
        enable = false
        remote_server = foo.bar.com
        remote_user = root
        remote_dir = /home/weewx/loop-data
        compress = False
        log_success = False
        ssh_options = "-o ConnectTimeout=1"
        timeout = 1
        skip_if_older_than = 3
    [[Include]]
        fields = dateTime, COMPASS_windDir, FMT_day_rain_total, FMT_dewpoint, FMT_outTemp, FMT_rainRate, FMT_windSpeed, FMT_HI_windGust, FMT_10mMaxGust, windSpeed
    [[Rename]]
        windRose = WindRose
```

## Entries in `LoopData` sections of `weewx.conf`:
 * `loop_data_dir`     : The directory into which the loop data file should be written.
 * `filename`          : The name of the loop data file to write.
 * `target_report`     : The WeeWX report to target.  LoopData will use this report to determine the
                         units to use and the formatting to apply.
 * `enable`            : Set to true to rsync the loop data file to `remote_server`.
 * `remote_server`     : The server to which gauge-data.txt will be copied.
                         To use rsync to sync loop-data.txt to a remote computer, passwordless ssh
                         using public/private key must be configured for authentication from the user
                         account that weewx runs under on this computer to the user account on the
                         remote machine with write access to the destination directory (remote_dir).
 * `remote_user`       : The userid on remote_server with write permission to remote_server_dir.
 * `remote_directory`  : The directory on remote_server where filename will be copied.
 * `compress`          : True to compress the file before sending.  Default is False.
 * `log_success`       : True to write success with timing messages to the log (for debugging).
                         Default is False.
 * `ssh_options`       : ssh options Default is '-o ConnectTimeout=1' (When connecting, time out in
                       1 second.)
 * `timeout`           : I/O timeout. Default is 1.  (When sending, timeout in 1 second.)
 * `skip_if_older_than`: Don't bother to rsync if greater than this number of seconds.  Default is 4.
                         (Skip this and move on to the next if this data is older than 4 seconds.
 * `fields`            : Used to specify which fields to include in the file.  If fields is missing
                         and Rename (see below) is also missing, all fields are included.
 * `Rename`            : Used to specify which fields to include and which names should be used as
                         keys (i.e., what these fields should be renamed.  If neither Rename nor fields
                         is specified, all fields are included.

## List of all fields available:

This section lists some likely fields.  In reality, LoopData runs through the daily
summaries and includes all observations it finds.  The daily summaries also
conveniently privide today's high (HI_<obs>), today's low (LO_<obs>),
today's sum (SUM_<obs>), today's average (AVG_<obs>) and today's weighted average
(WAVG_<obj> observations.

In addition to these observations, LoopData also tracks 10 min. max gust,
barometer rate and windrose data.

When no fields are specified, all fields will written to loop-data.txt.  **Thus, to get
a full list of fields,  comment out the fields line in the LoopData section of weewx.conf.
Next, look in the loop-data.txt file to find all of the available fields.**

 * `dateTime`          : The time of this loop packet (seconds since the epoch).
 * `usUnits`           : The units system all obeservations are expressed in.
                         This will be the unit system of the report specified by
                         target_report in weewx.conf.
 * `outTemp`           : Outside temperature.
 * `inTemp`            : Inside temperature.
 * `outHumidity`       : Outside humidity.
 * `pressure`          : Pressure
 * `windSpeed`         : Wind Speed
 * `windDir`           : Wind Direction
 * `windGust`          : Wind Gust (high wind speed)
 * `windGustDir`       : Wind Gust Direction
 * `day_rain_total`    : Total rainfall today. If your driver doesn't report this, use SUM_rain (or FMT_SUM_rain) instead.
 * `rain`              : Rain
 * `altimeter`         : Altimeter
 * `appTemp`           : Apparent Temperature Outside
 * `barometer`         : Barometer
 * `beaufort`          : Beaufort Wind Scale rating
 * `cloudbase`         : Cloudbase Elevation
 * `dewpoint`          : Dew Point
 * `heatindex`         : Heat Index
 * `humidex`           : Humidity Index
 * `maxSolarRad`       : Maximum Solar Radiation
 * `rainRate`          : Rate of Rain
 * `windchill`         : Wind Chill Factor
 * `FMT_<obs>`         : The above observations expressed as a formatted value, including
                         the units (e.g., '4.8 mph').
 * `LABEL_<obs>`       : The label for the units associted with the observation (e.g., 'mph').
                         This label also applies to the high and low fields for this observation.
 * `UNITS_<obs>`       : The units that the observation is expressed in.  Also the units
                         for the corresponding HI and LO entries.  Example: 'mile_per_hour'.
 * `LO_<obs>`          : The minimum value of the observation today.
 * `FMT_LO_<obs>`      : The low observation expressed as a formatted value, including
                         the units (e.g., '4.8 mph').
 * `T_LO<obs>`         : The time of the daily minimum observation.
 * `HI_<obs>`          : The maximum value of the observation today.
 * `FMT_HI_<obs>`      : The high observation expressed as a formatted value, including
                         the units (e.g., '4.8 mph').
 * `T_HI<obs>`         : The time of the daily maximum observation.
 * `SUM_<obs>`         : The sum of the observation values reported today.  SUM_rain is
                         especially useful for reporting the day's rain.
 * `FMT_SUM_<obs>`     : The sum of the observation values reported today.  It is formatted
                         and includes units (e.g., 0.42 in).
 * `AVG_<obs>`         : The average of the observation values reported today.
 * `FMT_AVG_<obs>`     : The average of the observation values reported today.  It is formatted
                         and includes units (e.g., 3.4 mph).
 * `WAVG_<obs>`        : The weighted average of the observation values reported today.
 * `FMT_WAVG_<obs>`    : The weighted average of the observation values reported today.  It is
                         formatted and includes units (e.g., 3.4 mph).
 * `COMPASS_<obs>`     : For windDir and windGustDir, text expression for the direction
                         (.e., 'NNE').
 * `10mMaxGust`        : The maximum wind gust in the last 10m.
 * `T_10mMaxGust`      : The time of the max gust (seconds since the epoch).
 * `FMT_10mMaxGust`    : 10mMaxGust expressed as a formatted value ('8.6 mph').
 * `LABEL_10mMaxGust`  : The label of the units for 10mMaxGust (e.g., 'mph').
 * `UNITS_10mMaxGust`  : The units that 10mMaxGust is expressed in (e.g., 'mile_per_hour').
 * `barometerRate`     : The difference in barometer in the last 3 hours
                         (i.e., barometer_3_hours_ago - barometer_now)
 * `DESC_barometerRate`: Shipping forecast descriptions for the 3 hour change in
                         barometer readings (e.g., "Falling Slowly').
 * `FMT_barometerRate`:  Formatted baromter rate (e.g., '0.2 inHg/h').
 * `UNITS_barometerRate`:The units used in baromter rate (e.g., 'inHg_per_hour').
                         barometer readings (e.g., "Falling Slowly').
 * `LABEL_barometerRate`:The label used for baromter rate units (e.g., 'inHg/hr').
 * `windRose`          : An array of 16 directions (N,NNE,NE,ENE,E,ESE,SE,SSE,S,SSW,SW,
                         WSW,W,WNW,NW,NNW) containing the distance traveled in each
                         direction.)
 * `LABEL_windRose`    : The label of the units for windRose (e.g., 'm')
 * `UNITS_windRose`    : The units that windrose values are expressed in (e.g., 'mile').

## About those Rsync Errors in the Log
If one is using rsync, especially if the loop interval is short (e.g., 2s), it is expected that
there will be log entries for connection timeouts, transmit timeouts, write errors and skipped
packets.  By default only one second is allowed to connect or transmit the data.  Also, by
default, if the loop data is older than 3s, it is skipped.  With these settings, the remote
server may miss receiving some loop-data packets, but it won't get caught behind trying to send
a backlog of old loop data.

Following are examples of a connection timeout, transmission timeout, writer error and a skipped
packet.  These errors are fine in moderation.  If too many packets are timing out, one might try
changing the connection timeout or timeout values.
```
Jul  1 04:12:03 charlemagne weewx[1126] ERROR weeutil.rsyncupload: [['rsync', '--archive', '--stats', '--timeout=1', '-e ssh -o ConnectTimeout=1', '/home/weewx/gauge-data/loop-data.txt', 'root@www.paloaltoweather.com:/home/weewx/gauge-data/loop-data.txt']] reported errors: ssh: connect to host www.paloaltoweather.com port 22: Connection timed out. rsync: connection unexpectedly closed (0 bytes received so far) [sender]. rsync error: unexplained error (code 255) at io.c(235) [sender=3.1.3]
Jun 30 20:51:48 charlemagne weewx[1126] ERROR weeutil.rsyncupload: [['rsync', '--archive', '--stats', '--timeout=1', '-e ssh -o ConnectTimeout=1', '/home/weewx/gauge-data/loop-data.txt', 'root@www.paloaltoweather.com:/home/weewx/gauge-data/loop-data.txt']] reported errors: [sender] io timeout after 1 seconds -- exiting. rsync error: timeout in data send/receive (code 30) at io.c(204) [sender=3.1.3]
Jun 27 10:18:37 charlemagne weewx[17982] ERROR weeutil.rsyncupload: [['rsync', '--archive', '--stats', '--timeout=1', '-e ssh -o ConnectTimeout=1', '/home/weewx/gauge-data/loop-data.txt', 'root@www.paloaltoweather.com:/home/weewx/gauge-data/loop-data.txt']] reported errors: rsync: [sender] write error: Broken pipe (32). rsync error: error in socket IO (code 10) at io.c(829) [sender=3.1.3]
Jun 27 23:15:53 charlemagne weewx[10156] INFO user.loopdata: skipping packet (2020-06-27 23:15:50 PDT (1593324950)) with age: 3.348237
```

## Why require Python 3?

LoopData is a new extension.  The author believes software written after Python 2 end of life
should not target Python 2.

## Licensing

weewx-loopdata is licensed under the GNU Public License v3.
