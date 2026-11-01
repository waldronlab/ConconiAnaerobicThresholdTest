# Analysis of Garmin TCX data for Conconi anaerobic threshold test

## Background

Rationale and validation for this method of estimating anaerobic
threshold is described by Conconi et al. (1996), but also disputed
e.g. by Jeukendrup et al. (1997) and Hofmann et al. (1997) (see
[References](#references)).

## What this software does

This software provides two functions:

1.  [`prepdata()`](https://lwaldron.github.io/ConconiAnaerobicThresholdTest/reference/prepdata.md)
    reads in a TCX file exported from Garmin Connect, Polar Flow, and
    probably other fitness apps that record heart rate. It also provides
    settings for the test, such as the start and end time, and the speed
    and time increments.
2.  [`fitmodel()`](https://lwaldron.github.io/ConconiAnaerobicThresholdTest/reference/fitmodel.md)
    fits a changepoint model with two linear regression lines that
    intersect at the estimated anaerobic threshold.

## How to use this software

You can either install it as an R package from CRAN, or use it as an
interactive web app at
<https://leviwaldron.shinyapps.io/ConconiAnaerobicThresholdTest/>.

## Performing the test

Use a chest strap heart rate monitor if available.

1.  set the treadmill to 1% incline to compensate for lack of wind
    resistance
2.  warm up by jogging on the treadmill at a slow speed for 10 minutes
3.  rest for 10 minutes with light static or dynamic stretching
4.  start at a low speed where you can still jog, not run, and have
    several steps before you hit your anaerobic threshold. 6km/h is a
    likely choice. km/h is likely better than mph for the smaller steps
    in speed.
5.  spend 90 seconds at each speed. After 90 seconds, increase the speed
    by 1 km/h
6.  continue until exhaustion, when you can no longer maintain the
    speed. The test ends and the end of the last segment where the full
    90 seconds are completed.

At the end of the test, you can remain still for several minutes with
the heart rate monitor still recording, to separately measure and
compare rate of recovery. Analysis of heart rate recovery isn’t
currently handled by this package however.

## Exporting the Garmin TCX file

Open the workout in the web version of [Garmin
Connect](https://connect.garmin.com). Click the gearbox in the upper
right-hand side:

![](Screenshot-wideangle.png)

Then, export to TCX:

![](Screenshot-menu.png)

## Non-Garmin data

Actually you don’t need to import a TCX file, what matters for the
[`fitmodel()`](https://lwaldron.github.io/ConconiAnaerobicThresholdTest/reference/fitmodel.md)
function is that you provide a data.frame with columns `time`,
`heart_rate`, and optionally `speed`.

If you import a TCX file that is not from Garmin Connect, you may need
to rename the column containing heart rate to `heart_rate` and the
column containing time to `time`. The `time` column should be in seconds
or a format that can be coerced to seconds using
[`as.numeric()`](https://rdrr.io/r/base/numeric.html), such as the
POSIXct/POSIXlt formats that most services likely provide. If
`useDeviceSpeed` is FALSE, then the speed column should be `speed`.

## How to figure out startminutes and endminutes

Useful in case the run was started before the start of the actual step
test, or ended after.

``` r

library(ConconiAnaerobicThresholdTest)
```

Note, files in this package are gzipped to save space. TCX files
exported as above will not have the `.gz` extension and you should not
use the [`gzfile()`](https://rdrr.io/r/base/connections.html) adapter
used below.

``` r

fname = system.file("extdata/2023-01-16.tcx.gz", package = "ConconiAnaerobicThresholdTest")
tmp <- prepdata(gzfile(fname), startminutes = 0, endminutes = 100,
          useDeviceSpeed = TRUE)
plot(tmp$minutes, tmp$speed)
```

![](usage_files/figure-html/unnamed-chunk-2-1.png)

``` r

plot(tmp$minutes, tmp$cadence_running)
```

![](usage_files/figure-html/unnamed-chunk-2-2.png)

``` r

plot(tmp$minutes, tmp$heart_rate)
```

![](usage_files/figure-html/unnamed-chunk-2-3.png)

By iteratively adjusting the `startminutes` and `endminutes` and
replotting, or just replotting with adjusted axes, I found that the
correct start time was at 0.15 minutes and correct end time at 15
minutes.

## January 2023 test

Import the data and show the same plots.

``` r

dat202301 <- prepdata(gzfile(fname), startminutes = 0.15, endminutes = 15,
          useDeviceSpeed = FALSE)
(dat202301$date = substr(dat202301$time[1], 1, 10))
#> [1] "2023-01-16"
```

``` r

plot(dat202301$minutes, dat202301$speed)
```

![](usage_files/figure-html/unnamed-chunk-4-1.png)

``` r

plot(dat202301$minutes, dat202301$cadence_running)
```

![](usage_files/figure-html/unnamed-chunk-4-2.png)

``` r

plot(dat202301$minutes, dat202301$heart_rate)
```

![](usage_files/figure-html/unnamed-chunk-4-3.png)

### Model fitting

This model uses all available data points:

``` r

fitmodel(dat202301, alldata = TRUE, title = "January 2023, using all HR data")
#> [1] "Threshold alpha: 11.8861748818123"
#> [1] ""
#> [1] "Model coefficients: Beta[0], Beta[1], Beta[2]"
#> (Intercept)           x           w 
#>   63.027275    9.167789   -5.201732
#> 
```

![](usage_files/figure-html/unnamed-chunk-5-1.png)

And this model uses only the final 5 measurements in each step:

``` r

fitmodel(dat202301, alldata = FALSE, title = "January 2023, using only last 5 HR measurements of each step")
#> [1] "Threshold alpha: 12.3667609360642"
#> [1] ""
#> [1] "Model coefficients: Beta[0], Beta[1], Beta[2]"
#> (Intercept)           x           w 
#>   73.442849    8.242858   -4.842851
#> 
```

![](usage_files/figure-html/unnamed-chunk-6-1.png)

## Sept 15, 2023 test

``` r

fname = system.file("extdata/2023-09-15.tcx.gz", 
                    package = "ConconiAnaerobicThresholdTest")
dat202309 <- prepdata(gzfile(fname), startminutes = 23.8, endminutes = 40.1,
          useDeviceSpeed = FALSE)
dat202309$date = substr(dat202309$time[1], 1, 10)
with(dat202309, plot(minutes, speed))
```

![](usage_files/figure-html/unnamed-chunk-7-1.png)

### Model fitting

``` r

fitmodel(dat202309, alldata = TRUE, title = "September 2023, using all HR data")
#> [1] "Threshold alpha: 14.5862765638889"
#> [1] ""
#> [1] "Model coefficients: Beta[0], Beta[1], Beta[2]"
#> (Intercept)           x           w 
#>   76.921892    7.205029   -4.961977
#> 
```

![](usage_files/figure-html/unnamed-chunk-8-1.png)

``` r

fitmodel(dat202309, alldata = FALSE, title = "September 2023, using only last 5 HR measurements of each step")
#> [1] "Threshold alpha: 14.3405968989866"
#> [1] ""
#> [1] "Model coefficients: Beta[0], Beta[1], Beta[2]"
#> (Intercept)           x           w 
#>   82.822218    6.880000   -4.279986
#> 
```

![](usage_files/figure-html/unnamed-chunk-8-2.png)

## Jan 10, 2022 test

``` r

fname = system.file("extdata/2022-01-10.tcx.gz",
                    package = "ConconiAnaerobicThresholdTest")
dat202201 <- prepdata(gzfile(fname), startminutes = 26, endminutes = 38.99,
          useDeviceSpeed = FALSE)
dat202201$date = substr(dat202201$time[1], 1, 10)
```

### Model fitting

``` r

fitmodel(dat202201, alldata = TRUE, title = "January 2022, using all HR data")
#> [1] "Threshold alpha: 10.4210375092318"
#> [1] ""
#> [1] "Model coefficients: Beta[0], Beta[1], Beta[2]"
#> (Intercept)           x           w 
#>   76.397735    9.484450   -4.844825
#> 
```

![](usage_files/figure-html/unnamed-chunk-10-1.png)

``` r

fitmodel(dat202201, alldata = FALSE, title = "January 2022, using only last 5 HR measurements of each step")
#> [1] "Threshold alpha: 9.63106140240337"
#> [1] ""
#> [1] "Model coefficients: Beta[0], Beta[1], Beta[2]"
#> (Intercept)           x           w 
#>   78.199938   10.080009   -6.180004
#> 
```

![](usage_files/figure-html/unnamed-chunk-10-2.png)

## 2022-23 together

Some plots demonstrating comparison of two tests. First join the two
data.frames, and convert date and speed to factors (to make plots appear
the way I want them to).

``` r

xall <- full_join(x=dat202309, y=dat202301) |>
    full_join(y=dat202201) |>
    mutate(date = factor(date)) |>
    mutate(speed = factor(speed))
#> Joining with `by = join_by(time, latitude, longitude, altitude, distance,
#> heart_rate, speed, cadence_running, cadence_cycling, power, temperature,
#> minutes, date)`
#> Joining with `by = join_by(time, latitude, longitude, altitude, distance,
#> heart_rate, speed, cadence_running, cadence_cycling, power, temperature,
#> minutes, date)`
```

### Boxplot of heart rate vs speed

``` r

ggplot(xall, aes(x=speed, y=heart_rate, fill = date)) +
    geom_boxplot()  +
    scale_y_continuous(breaks=seq(90, 200, by=10))
```

![](usage_files/figure-html/unnamed-chunk-12-1.png)

### Scatterplot with Loess smoothing curve for heart rate vs speed

``` r

ggplot(xall, aes(x = minutes, y = heart_rate, color = date)) +
    geom_point(size = 0.5)  +
    geom_smooth() +
    scale_y_continuous(breaks = seq(90, 200, by = 10), name = "Heart Rate (bpm)") +
    scale_x_continuous(breaks = seq(0, 16.5, by = 1.5), name = "Time (minutes)", 
                       sec.axis = sec_axis( ~ . / 1.5 + 6, 
                                            name = "speed (km/h)", 
                                            breaks = seq(6, 16, by = 1)))
#> `geom_smooth()` using method = 'loess' and formula = 'y ~ x'
```

![](usage_files/figure-html/unnamed-chunk-13-1.png)

### Scatterplot with Loess smoothing curve for running cadence vs speed

``` r

filter(xall, date != "2022-01-10") |> # bad cadence data from 2022-01-10
    ggplot(aes(
        x = as.numeric(as.character(speed)),
        y = 2 * cadence_running,
        color = date
    )) +
    geom_point(size = 0.5)  +
    geom_smooth() +
    scale_y_continuous(breaks = seq(150, 200, by = 10), name = "Cadence (spm)") +
    scale_x_continuous(
        breaks = seq(0, 16.5, by = 1),
        name = "speed (km/h)",
        sec.axis = sec_axis(~ . / 1.5 + 6,
                            name = "speed (km/h)",
                            breaks = seq(6, 16, by = 1))
    )
#> `geom_smooth()` using method = 'loess' and formula = 'y ~ x'
```

![](usage_files/figure-html/unnamed-chunk-14-1.png)

## References

Conconi, F, G Grazzi, I Casoni, et al. 1996. “The Conconi Test:
Methodology After 12 Years of Application.” *International Journal of
Sports Medicine* 17 (7): 509–19.
<https://doi.org/10.1055/s-2007-972887>.

Hofmann, P, R Pokan, S P von Duvillard, and P Schmid. 1997. “The Conconi
Test.” *International Journal of Sports Medicine* 18 (5): 397–99.
<https://doi.org/10.1055/s-2007-972654>.

Jeukendrup, A E, M K Hesselink, H Kuipers, and H A Keizer. 1997. “The
Conconi Test.” *International Journal of Sports Medicine* 18 (5):
393–96. <https://doi.org/10.1055/s-2007-972652>.
