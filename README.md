# d3-scale

Scales are a convenient abstraction for a fundamental task in visualization: mapping a dimension of abstract data to a visual representation. Although most often used for position-encoding quantitative data, such as mapping the height in meters of a sample population to the height in pixels of bars in a bar chart, scales can represent virtually any visual encoding, such as diverging colors, stroke widths, or symbol size. Scales can also be used with virtually any type of data, such as named categorical data or discrete data that needs sensible breaks.

For continuous quantitative data, you typically want a [linear scale](#linear-scales). (For time series data, a [time scale](#time-scales).) If the distribution calls for it, consider transforming data using a [power](#power-scales) or [log](#log-scales) scale. A [quantize scale](#quantize-scales) may aid differentiation by rounding continuous data to a fixed set of discrete values; similarly, a [quantile scale](#quantile-scales) computes quantiles from a sample population, and a [threshold scale](#threshold-scales) allows you to specify arbitrary breaks in continuous data. Several built-in [sequential color scales](#sequential-color-scales) are also provided. (If you don’t like these palettes, try [ColorBrewer](http://colorbrewer2.org/).)

For ordinal (ordered) or categorical data, an [ordinal scale](#ordinal-scales) specifies an explicit mapping from a discrete set of data values to a corresponding set of visual attributes (such as colors). The related [band](#band) and [point](#point) scales are useful for position-encoding ordinal data, such as bars in a bar chart or dots in an categorical scatterplot. Several built-in [categorical color scales](#categorical-color-scales) are also provided.

Scales have no intrinsic visual representation; for that, consider an [axis](https://github.com/mbostock/d3/wiki/SVG-Axes). However, scales typically provide a [ticks method](#linear_ticks) and a [tick format](#linear_tickFormat) that can be used to render reference marks.

For a longer introduction, see these recommended tutorials:

* [Chapter 7. Scales](http://chimera.labs.oreilly.com/books/1230000000345/ch07.html) of *Interactive Data Visualization for the Web* by Scott Murray

* [d3: scales, and color.](http://www.jeromecukier.net/blog/2011/08/11/d3-scales-and-color/) by Jérôme Cukier

## Installing

If you use NPM, `npm install d3-scale`. Otherwise, download the [latest release](https://github.com/d3/d3-scale/releases/latest). The released bundle supports AMD, CommonJS, and vanilla environments. Create a custom build using [Rollup](https://github.com/rollup/rollup) or your preferred bundler. You can also load directly from [d3js.org](https://d3js.org):

```html
<script src="https://d3js.org/d3-array.v0.6.min.js"></script>
<script src="https://d3js.org/d3-color.v0.3.min.js"></script>
<script src="https://d3js.org/d3-format.v0.4.min.js"></script>
<script src="https://d3js.org/d3-interpolate.v0.3.min.js"></script>
<script src="https://d3js.org/d3-time.v0.1.min.js"></script>
<script src="https://d3js.org/d3-time-format.v0.2.min.js"></script>
<script src="https://d3js.org/d3-scale.v0.3.min.js"></script>
```

(If you’re not using [time scales](#time), you can omit d3-time and d3-time-format.) In a vanilla environment, a `d3_scale` global is exported. [Try d3-scale in your browser.](https://tonicdev.com/npm/d3-scale)

## API Reference

* [Quantitative](#quantitative-scales)
* [Quantize](#quantize-scales)
* [Quantile](#quantile-scales)
* [Threshold](#threshold-scales)
* [Identity](#identity-scales)
* [Sequential Color](#sequential-color-scales)
* [Ordinal](#ordinal-scales)
* [Categorical Color](#categorical-color-scales)

### Quantitative Scales

Quantitative scales map a continuous, numeric input [domain](#quantitative_domain) to a continuous output [range](#quantitative_range).

<a name="quantitative" href="#quantitative">#</a> <i>quantitative</i>(<i>value</i>)

Given a *value* in the [domain](#quantitative_domain), returns the corresponding value from the [range](#quantitative_range). If the given *value* is outside the domain, and [clamping](#quantitative_clamp) is not enabled, the quantitative mapping may be extrapolated such that the returned value is outside the range. For example, to apply a position encoding:

```js
var x = d3_scale.linear()
    .domain([10, 130])
    .range([0, 960]);

x(20); // 80
x(50); // 320
```

<a name="quantitative_invert" href="#quantitative_invert">#</a> <i>quantitative</i>.<b>invert</b>(<i>value</i>)

Given a *value* in the [range](#quantitative_range), returns the corresponding value from the [domain](#quantitative_domain). Inversion is useful for interaction, say to determine the value in the domain that corresponds to the position under the mouse. If the given *value* is outside the range, and [clamping](#quantitative_clamp) is not enabled, the quantitative mapping may be extrapolated such that the returned value is outside the down. This method is only supported if the range is numeric. If the range is not numeric, returns NaN. For example, to invert a position encoding:

```js
var x = d3_scale.linear()
    .domain([10, 130])
    .range([0, 960]);

x.invert(80); // 20
x.invert(320); // 50
```

For a valid value *y* in the range, <i>quantitative</i>(<i>quantitative</i>.invert(<i>y</i>)) approximately equals *y*; similarly, for a valid value *x* in the domain, <i>quantitative</i>.invert(<i>quantitative</i>(<i>x</i>)) approximately equals *x*. (The scale and its inverse may not be exact due to the limitations of floating point precision.)

<a name="quantitative_domain" href="#quantitative_domain">#</a> <i>quantitative</i>.<b>domain</b>([<i>domain</i>])

If *domain* is specified, sets the scale’s domain to the specified array of numbers. The array must contain two or more elements. If the elements in the given array are not numbers, they will be coerced to numbers. If *domain* is not specified, returns the scale’s current domain.

Although quantitative scales typically have two values each in their domain and range, specifying more than two values produces a piecewise quantitative scale. For example, to create a diverging color scale that interpolates between white and red for negative values, and white and green for positive values, say:

```js
var color = d3_scale.linear()
    .domain([-1, 0, 1])
    .range(["red", "white", "green"]);

color(-0.5); // "#ff8080"
color(+0.5); // "#80c080"
```

Internally, a quantitative piecewise scale performs a [binary search](https://github.com/d3/d3-array#bisect) for the range interpolator corresponding to the given domain value. Thus, the domain must be in ascending or descending order. If the domain and range have different lengths *N* and *M*, only the first *min(N,M)* elements in each are observed.

<a name="quantitative_range" href="#quantitative_range">#</a> <i>quantitative</i>.<b>range</b>([<i>range</i>])

If *range* is specified, sets the scale’s range to the specified array of values. The array must contain two or more elements. Unlike the [domain](#quantitative_domain), elements in the given array need not be numbers; any value that is supported by the underlying [interpolator](#quantitative_interpolate) will work. For example, to apply a sequential color encoding:

```js
var color = d3_scale.linear()
    .domain([10, 100])
    .range(["brown", "steelblue"]);

color(20); // "#9a3439"
color(50); // "#7b5167"
```

See [*quantitative*.interpolate](#quantitative_interpolate) for more examples. Note that numeric ranges are required for [invert](#quantiative_invert).

If *range* is not specified, returns the scale’s current range.

<a name="quantitative_rangeRound" href="#quantitative_rangeRound">#</a> <i>quantitative</i>.<b>rangeRound</b>([<i>range</i>])

Sets the scale’s [*range*](#quantitative_range) to the specified array of values while also setting the scale’s [interpolator](#quantitative_interpolate) to d3-interpolate’s [round](https://github.com/d3/d3-interpolate#round). This is a convenience method equivalent to:

```js
quantitative
    .range(range)
    .interpolate(d3_interpolate.round);
```

The rounding interpolator is sometimes useful for avoiding antialiasing artifacts, though also consider the [shape-rendering](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/shape-rendering) “crispEdges” styles. Note that this interpolator can only be used with numeric ranges.

<a name="quantitative_clamp" href="#quantitative_clamp">#</a> <i>quantitative</i>.<b>clamp</b>(<i>clamp</i>)

If *clamp* is specified, enables or disables clamping accordingly. If clamping is disabled and the scale is passed a value outside the [domain](#quantitative_domain), the scale may return a value outside the [range](#quantitative_range) through extrapolation. If clamping is enabled, the return value of the scale is always within the scale’s range. Clamping similarly applies to [*quantitative*.invert](#quantitative_invert). For example:

```js
var x = d3_scale.linear()
    .domain([10, 130])
    .range([0, 960]);

x(-10); // -160
x.invert(-160); // -10

x.clamp(true);
x(-10); // 0
x.invert(-160); // 10
```

If *clamp* is not specified, returns whether or not the scale currently clamps values to within the range.

<a name="quantitative_interpolate" href="#quantitative_interpolate">#</a> <i>quantitative</i>.<b>interpolate</b>(<i>interpolate</i>[, <i>parameters…</i>])

If *interpolate* is specified, sets the scale’s [range](#quantitative_range) interpolator factory. This interpolator factory is used to create interpolators for each adjacent pair of values from the range; these interpolators then map a normalized domain parameter *t* in [0, 1] to the corresponding value in the range. If *factory* is not specified, returns the scale’s current interpolator factory, which defaults to d3-interpolate’s [value](https://github.com/d3/d3-interpolate#value).

For example, consider a diverging color scale with three colors in the range:

```js
var color = d3_scale.linear()
    .domain([-100, 0, +100])
    .range(["red", "white", "green"]);
```

Two interpolators are created internally by the scale, equivalent to:

```js
var i0 = d3_interpolate.value("red", "white"),
    i1 = d3_interpolate.value("white", "green");
```

A common reason to specify a custom interpolator is to change the color space of interpolation. For example, to use the [HCL color space](https://github.com/d3/d3-interpolate#hcl):

```js
var color = d3_scale.linear()
    .domain([10, 100])
    .range(["brown", "steelblue"])
    .interpolate(d3_interpolate.hcl);
```

See [d3-interpolate](https://github.com/d3/d3-interpolate) for more interpolators. Note: the default interpolator, [value](https://github.com/d3/d3-interpolate#value), **may reuse return values**. For example, if the range values are objects, then the default interpolator always returns the same object, modifying it in-place. If the scale is used to set an attribute or style, this is typically acceptable; however, if you need to store the scale’s return value, you must specify your own interpolator or make a copy as appropriate.

<a name="quantitative_ticks" href="#quantitative_ticks">#</a> <i>quantitative</i>.<b>ticks</b>([<i>count</i>])

Returns approximately *count* representative values from the scale’s [domain](#quantitative_domain). If *count* is not specified, it defaults to 10. The returned tick values are uniformly spaced, have human-readable values (such as multiples of powers of 10), and are guaranteed to be within the extent of the domain. Ticks are often used to display reference lines, or tick marks, in conjunction with the visualized data. The specified *count* is only a hint; the scale may return more or fewer values depending on the domain. See also d3-array’s [ticks](https://github.com/d3/d3-array#ticks).

<a name="quantitative_tickFormat" href="#quantitative_tickFormat">#</a> <i>quantitative</i>.<b>tickFormat</b>([<i>count</i>[, <i>specifier</i>]])

Returns a [number format](https://github.com/d3/d3-format) function suitable for displaying a tick value, automatically computing the appropriate precision based on the fixed interval between tick values. The specified *count* should have the same value as the count that is used to generate the [tick values](#quantitative_ticks).

The optional *specifier* argument allows a [custom format](https://github.com/d3/d3-format#locale_format) where the precision of the format is automatically substituted by the scale to be appropriate for the tick interval. For example, to format percentage change, you might say:

```js
var x = d3_scale.linear()
    .domain([-1, 1])
    .range([0, 960]);

var ticks = x.ticks(5),
    tickFormat = x.tickFormat(5, "+%");

ticks.map(tickFormat); // ["-100%", "-50%", "+0%", "+50%", "+100%"]
```

If *specifier* uses the format type `s`, the scale will return a [SI-prefix format](https://github.com/d3/d3-format#locale_formatPrefix) based on the largest value in the domain. If the *specifier* already specifies a precision, this method is equivalent to [*locale*.format](https://github.com/d3/d3-format#locale_format).

<a name="quantitative_nice" href="#quantitative_nice">#</a> <i>quantitative</i>.<b>nice</b>([<i>count</i>])

Extends the [domain](#quantitative_domain) so that it starts and ends on nice round values. This method typically modifies the scale’s domain, and may only extend the bounds to the nearest round value. An optional tick *count* argument allows greater control over the step size used to extend the bounds, guaranteeing that the returned [ticks](#quantitative_ticks) will exactly cover the domain. Nicing is useful if the domain is computed from data, say using [extent](https://github.com/d3/d3-array#extent), and may be irregular. For example, for a domain of [0.201479…, 0.996679…], a nice domain might be [0.2, 1.0]. If the domain has more than two values, nicing the domain only affects the first and last value. See also d3-array’s [tickStep](https://github.com/d3/d3-array#tickStep).

<a name="quantitative_copy" href="#quantitative_copy">#</a> <i>quantitative</i>.<b>copy</b>()

Returns an exact copy of this scale. Changes to this scale will not affect the returned scale, and vice versa.

<a name="linear" href="#linear">#</a> <b>linear</b>()

Constructs a new linear scale with the unit [domain](#quantitative_domain) [0, 1], the unit [range](#quantitative_range) [0, 1], a [value](https://github.com/d3/d3-interpolate#value) [interpolator](#quantitative_interpolate) and [clamping](#quantitative_clamp) disabled. Linear scales are a good default choice for quantitative data because they preserve proportional differences. Each range value *y* can be expressed as a function of the domain value *x*: *y* = *mx* + *b*.

<a name="pow" href="#pow">#</a> <b>pow</b>()

Constructs a new power scale with the unit [domain](#quantitative_domain) [0, 1], the unit [range](#quantitative_range) [0, 1], the [exponent](#pow_exponent) 1, a [value](https://github.com/d3/d3-interpolate#value) [interpolator](#quantitative_interpolate) and [clamping](#quantitative_clamp) disabled. (Note that this is effectively a [linear](#linear) scale until you set a different exponent.) Power scales are similar to [linear scales](#linear), except an exponential transform is applied to the input domain value before the output range value is computed. Each range value *y* can be expressed as a function of the domain value *x*: *y* = *mx^k* + *b*, where *k* is the [exponent](#pow_exponent) value. Power scales also support negative domain values, in which case the input value and the resulting output value are multiplied by -1.

<a name="pow_exponent" href="#pow_exponent">#</a> <i>pow</i>.<b>exponent</b>([<i>exponent</i>])

If *exponent* is specified, sets the current exponent to the given numeric value. If *exponent* is not specified, returns the current exponent, which defaults to 1. (Note that this is effectively a [linear](#linear) scale until you set a different exponent.)

<a name="sqrt" href="#sqrt">#</a> <b>sqrt</b>()

Constructs a new [power scale](#pow) with the unit [domain](#quantitative_domain) [0, 1], the unit [range](#quantitative_range) [0, 1], the [exponent](#pow_exponent) 0.5, a [value](https://github.com/d3/d3-interpolate#value) [interpolator](#quantitative_interpolate) and [clamping](#quantitative_clamp) disabled. This is a convenience method equivalent to `d3_scale.pow().exponent(0.5)`.

<a name="log" href="#log">#</a> <b>log</b>()

Constructs a new log scale with the [domain](#quantitative_domain) [1, 10], the unit [range](#quantitative_range) [0, 1], the [base](#log_base) 10, a [value](https://github.com/d3/d3-interpolate#value) [interpolator](#quantitative_interpolate) and [clamping](#quantitative_clamp) disabled. Log scales are similar to [linear scales](#linear), except a logarithmic transform is applied to the input domain value before the output range value is computed. The mapping to the range value *y* can be expressed as a function of the domain value *x*: *y* = *m* log(<i>x</i>) + *b*.

As log(0) = -∞, a log scale domain must be **strictly-positive or strictly-negative**; the domain must not include or cross zero. A log scale with a positive domain has a well-defined behavior for positive values, and a log scale with a negative domain has a well-defined behavior for negative values. (For a negative domain, input and output values are implicitly multiplied by -1.) The behavior of the scale is undefined if you pass a negative value to a log scale with a positive domain or vice versa.

<a name="log_base" href="#log_base">#</a> <i>log</i>.<b>base</b>([<i>base</i>])

If *base* is specified, sets the base for this logarithmic scale to the specified value. If *base* is not specified, returns the current base, which defaults to 10.

<a name="log_nice" href="#log_nice">#</a> <i>log</i>.<b>nice</b>()

Like [*quantitative*.nice](#quantitative_nice), except extends the domain to integer powers of [base](#log_base). For example, for a domain of [0.201479…, 0.996679…], the nice domain is [0.1, 1]. If the domain has more than two values, nicing the domain only affects the first and last value.

<a name="log_ticks" href="#log_ticks">#</a> <i>log</i>.<b>ticks</b>([<i>count</i>])

Like [*quantitative*.ticks](#quantitative_ticks), but customized for a log scale. If the [base](#log_base) is an integer, the returned ticks are uniformly spaced within each integer power of base; otherwise, one tick per power of base is returned. The returned ticks are guaranteed to be within the extent of the domain. If the orders of magnitude in the [domain](#quantitative_domain) is greater than *count*, then at most one tick per power is returned. Otherwise, the tick values are unfiltered, but note that you can use [*log*.tickFormat](#log_tickFormat) to filter the display of tick lables. If *count* is not specified, it defaults to 10.

<a name="log_tickFormat" href="#log_tickFormat">#</a> <i>log</i>.<b>tickFormat</b>([<i>count</i>[, <i>specifier</i>]])

Like [*quantitative*.tickFormat](#quantitative_tickFormat), but customized for a log scale. If a *count* is specified, then the formatter may return the empty string for some of the tick labels; this is useful if there is not enough room to fit all of the tick labels, but you still want to display the tick marks to show the log scale distortion. When specifying a count, you may also provide a format *specifier* or format function. For example, to get a tick formatter that will display 20 ticks of a currency, say `log.tickFormat(20, "$,f")`. If the specifier does not have a defined precision, the precision will be set automatically by the scale, returning the appropriate format. This provides a convenient, declarative way of specifying a format whose precision will be automatically set by the scale.

CAUTION: DOCUMENTATION BELOW THIS POINT IS BEING REWRITTEN.

### Time Scales

Time scales are a variant of [linear scales](#linear-scales) that have a time domain: domain values are coerced to [dates](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date) rather than numbers, and [*time*.invert](#time_invert) likewise returns a date. More importantly, time scales implement [ticks](#time_ticks) based on [conventional time intervals](https://github.com/d3/d3-time), taking the pain out of generating axes for temporal domains.

<a name="time" href="#time">#</a> <b>time</b>()

Constructs a new time scale with the default [domain](#time_domain) [2000-01-01,2000-01-02], the default [range](#time_range) [0, 1], the default [interpolator](#time_interpolate) and [clamping](#time_clamp) disabled.

<a name="_time" href="#_time">#</a> <i>time</i>(<i>date</i>)

Given a *date* in the [domain](#time_domain), returns the corresponding value in the [range](#time_range). For example, a position encoding:

```js
var s = time()
    .domain([new Date(2000, 0, 1), new Date(2000, 0, 2)])
    .range([0, 960]);

s(new Date(2000, 0, 1,  5)); // 200
s(new Date(2000, 0, 1, 16)); // 640
```

<a name="time_invert" href="#time_invert">#</a> <i>time</i>.<b>invert</b>(<i>y</i>)

Given a value *y* in the [range](#time_range), returns the corresponding date in the [domain](#time_domain): the inverse of [*time*](#_time). For example, a position encoding:

```js
var s = time()
    .domain([new Date(2000, 0, 1), new Date(2000, 0, 2)])
    .range([0, 960]);

s.invert(200); // Sat Jan 01 2000 05:00:00 GMT-0800 (PST)
s.invert(640); // Sat Jan 01 2000 16:00:00 GMT-0800 (PST)
```

This method is only supported if the range is numeric, and may return undefined if the range is non-numeric (such as colors). For a valid value *y* in the range, <i>time</i>(<i>time</i>.invert(<i>y</i>)) equals *y*; similarly, for a valid value *x* in the domain, <i>time</i>.invert(<i>time</i>(<i>x</i>)) equals *x*. The invert method is useful for interaction, say to determine the value in the domain that corresponds to the pixel location under the mouse.

<a name="time_domain" href="#time_domain">#</a> <i>time</i>.<b>domain</b>([<i>dates</i>])

If *domain* is specified, sets the scale’s domain to the specified array of dates. The array must contain two or more dates. If the elements in the given array are not dates, they will be coerced to dates. If *domain* is not specified, returns the scale’s current domain.

As with [*linear*.domain](#linear_domain), this method can accept more than two values for the domain and range, thus resulting in a “polylinear” scale.

<a name="time_range" href="#time_range">#</a> <i>time</i>.<b>range</b>([<i>range</i>])

If *range* is specified, sets the scale’s range to the specified array of values. The array must contain two or more values, matching the cardinality of the [domain](#range_domain); otherwise, the longer of the two is truncated to match the other. The elements in the given array need not be numbers; any value that is supported by the underlying [interpolator](#range_interpolate) will work; however, numeric ranges are required for [invert](#range_invert). If *range* is not specified, returns the scale’s current range.

<a name="time_rangeRound" href="#time_rangeRound">#</a> <i>time</i>.<b>rangeRound</b>([<i>range</i>])

Sets the scale’s [*range*](#time_range) to the specified array of values while also setting the scale’s [interpolator](#time_interpolate) to [interpolateRound](https://github.com/d3/d3-interpolate#interpolateRound). This is a convenience method equivalent to:

```js
s.range(range).interpolate(interpolateRound);
```

The rounding interpolator is sometimes useful for avoiding antialiasing artifacts, though also consider [shape-rendering](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/shape-rendering): crispEdges. Note that this interpolator can only be used with numeric ranges.

<a name="time_interpolate" href="#time_interpolate">#</a> <i>time</i>.<b>interpolate</b>([<i>interpolate</i>[, <i>parameters…</i>]])

If *interpolate* is specified, sets the scale’s [range](#time_range) interpolator factory. This interpolator factory is used to create interpolators for each adjacent pair of values from the range; these interpolators then map a normalized domain parameter *t* in [0, 1] to the corresponding value in the range. If *factory* is not specified, returns the scale’s interpolator factory. See [*linear*.interpolate](#linear_interpolate) for examples.

Note: the [default interpolator](https://github.com/d3/d3-interpolate#interpolate) **may reuse return values**. For example, if the domain values are arbitrary objects, then the default interpolator always returns the same object, modifying it in-place. If the scale is used to set an attribute or style, you typically don’t have to worry about this recyling of the scale’s return value; however, if you need to store the scale’s return value, specify your own interpolator or make a copy as appropriate.

<a name="time_clamp" href="#time_clamp">#</a> <i>time</i>.<b>clamp</b>([<i>clamp</i>])

If *clamp* is specified, enables or disables clamping accordingly. If clamping is disabled and the scale is passed a value outside the [domain](#time_domain), the scale may return a value outside the [range](#time_range) through extrapolation. If clamping is enabled, the return value of the scale is always within the scale’s range. Clamping similarly applies to [*time*.invert](#time_invert). See [*linear*.clamp](#linear_clamp) for examples. If *clamp* is not specified, returns whether or not the scale currently clamps values to within the range.

<a name="time_nice" href="#time_nice">#</a> <i>time</i>.<b>nice</b>([<i>count</i>])
<br><a name="time_nice" href="#time_nice">#</a> <i>time</i>.<b>nice</b>([<i>interval</i>[, <i>step</i>]])

Extends the [domain](#time_domain) so that it starts and ends on nice round values. This method typically modifies the scale’s domain, and may only extend the bounds to the nearest round value.

An optional tick *count* argument allows greater control over the step size used to extend the bounds, guaranteeing that the returned [ticks](#time_ticks) will exactly cover the domain. Alternatively, a [time *interval*](https://github.com/d3/d3-time#intervals) may be specified to explicitly set the ticks. If an *interval* is specified, an optional *step* may also be specified to skip some ticks. For example, `nice(second, 10)` will extend the domain to an even ten seconds (0, 10, 20, <i>etc.</i>). See [*time*.ticks](#time_ticks) and [*interval*.every](https://github.com/d3/d3-time#interval_every) for further detail.

Nicing is useful if the domain is computed from data, say using [extent](https://github.com/d3/d3-array#extent), and may be irregular. For example, for a domain of [2009-07-13T00:02, 2009-07-13T23:48], the nice domain is [2009-07-13, 2009-07-14]. If the domain has more than two values, nicing the domain only affects the first and last value.

<a name="time_ticks" href="#time_ticks">#</a> <i>time</i>.<b>ticks</b>([<i>count</i>])
<br><a name="time_ticks" href="#time_ticks">#</a> <i>time</i>.<b>ticks</b>([<i>interval</i>[, <i>step</i>]])

Returns representative dates from the scale’s [domain](#time_domain). The returned tick values are uniformly-spaced (mostly), have sensible values (such every day at midnight), and are guaranteed to be within the extent of the domain. Ticks are often used to display reference lines, or tick marks, in conjunction with the visualized data.

An optional *count* may be specified to affect how many ticks are generated. If *count* is not specified, it defaults to 10. The specified *count* is only a hint; the scale may return more or fewer values depending on the domain. For example, to create ten default ticks, say:

```js
var s = time();
s.ticks(10);
// [Sat Jan 01 2000 00:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 03:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 06:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 09:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 12:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 15:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 18:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 21:00:00 GMT-0800 (PST),
//  Sun Jan 02 2000 00:00:00 GMT-0800 (PST)]
```

The following time intervals are considered for automatic ticks:

* 1-, 5-, 15- and 30-second.
* 1-, 5-, 15- and 30-minute.
* 1-, 3-, 6- and 12-hour.
* 1- and 2-day.
* 1-week.
* 1- and 3-month.
* 1-year.

In lieu of a *count*, a [time *interval*](https://github.com/d3/d3-time#intervals) may be explicitly specified. If an *interval* is specified, an optional *step* may also be specified to prune generated ticks. For example, `ticks(minute, 15)` will generate ticks at 15-minute intervals:

```js
var s = time().domain([new Date(2000, 0, 1, 0), new Date(2000, 0, 1, 2)]);
s.ticks(minute, 15);
// [Sat Jan 01 2000 00:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 00:15:00 GMT-0800 (PST),
//  Sat Jan 01 2000 00:30:00 GMT-0800 (PST),
//  Sat Jan 01 2000 00:45:00 GMT-0800 (PST),
//  Sat Jan 01 2000 01:00:00 GMT-0800 (PST),
//  Sat Jan 01 2000 01:15:00 GMT-0800 (PST),
//  Sat Jan 01 2000 01:30:00 GMT-0800 (PST),
//  Sat Jan 01 2000 01:45:00 GMT-0800 (PST),
//  Sat Jan 01 2000 02:00:00 GMT-0800 (PST)]
```

This is equivalent to using [minute](https://github.com/d3/d3-time#minute).[every](https://github.com/d3/d3-time#interval_every)(15) or minute.filter with the following test function:

```js
s.ticks(minute.filter(function(d) { return d.getMinutes() % 15 === 0; }));
```

Note: in some cases, such as with day ticks, specifying a *step* can result in irregular spacing of ticks because months have varying length.

<a name="time_tickFormat" href="#time_tickFormat">#</a> <i>time</i>.<b>tickFormat</b>([<i>specifier</i>])

Returns a time format function suitable for displaying [tick](#time_ticks) values. If a format *specifier* is specified, this method is equivalent to [format](https://github.com/d3/d3-time-format#format). If *specifier* is not specified, the default time format is returned. The default multi-scale time format chooses a human-readable representation based on the specified date as follows:

* `%Y` - for year boundaries, such as `2011`.
* `%B` - for month boundaries, such as `February`.
* `%b %d` - for week boundaries, such as `Feb 06`.
* `%a %d` - for day boundaries, such as `Mon 07`.
* `%I %p` - for hour boundaries, such as `01 AM`.
* `%I:%M` - for minute boundaries, such as `01:23`.
* `:%S` - for second boundaries, such as `:45`.
* `.%L` - milliseconds for all other times, such as `.012`.

Although somewhat unusual, this default behavior has the benefit of providing both local and global context: for example, formatting a sequence of ticks as [11 PM, Mon 07, 01 AM] reveals information about hours, dates, and day simultaneously, rather than just the hours [11 PM, 12 AM, 01 AM]. See [d3-time-format](https://github.com/d3/d3-time-format) if you’d like to roll your own conditional time format.

<a name="time_copy" href="#time_copy">#</a> <i>time</i>.<b>copy</b>()

Returns an exact copy of this time scale. Changes to this scale will not affect the returned scale, and vice versa.

<a name="utcTime" href="#utcTime">#</a> <b>utcTime</b>()

Equivalent to [time](#time), but the returned time scale operates in [Coordinated Universal Time](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) rather than local time.

### Quantize Scales

Quantize scales are a variant of [linear scales](#linear-scales) with a discrete rather than continuous range. The input domain is still continuous, and divided into uniform segments based on the number of values in (the cardinality of) the output range. Each range value *y* can be expressed as a quantized linear function of the domain value *x*: *y* = *m round(x)* + *b*. See [bl.ocks.org/4060606](http://bl.ocks.org/mbostock/4060606) for an example.

<a name="quantize" href="#quantize">#</a> <b>quantize</b>()

Constructs a new quantize scale with the default [domain](#quantize_domain) [0, 1] and the default [range](#quantize_range) [0, 1]. Thus, the default quantize scale is equivalent to the [Math.round](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Math/round) function.

<a name="_quantize" href="#_quantize">#</a> <i>quantize</i>(<i>x</i>)

Given a value *x* in the input [domain](#quantize_domain), returns the corresponding value *y* in the output [range](#quantize_rangE). For example, a color encoding:

```js
var s = quantize().domain([0, 1]).range(["brown", "steelblue"]);
s(0.49); // "brown"
s(0.51); // "steelblue"
```

Dividing the domain into three equally-sized parts with different range values, say to compute an appropriate stroke width:

```js
var s = quantize().domain([10, 100]).range([1, 2, 4]);
s(20); // 1
s(50); // 2
s(80); // 4
```

<a name="quantize_invertExtent" href="#quantize_invertExtent">#</a> <i>quantize</i>.<b>invertExtent</b>(<i>y</i>)

Returns the extent of values in the [domain](#quantize_domain) [<i>x0</i>, <i>x1</i>] for the corresponding value in the [range](#quantize_range) *y*: the inverse of [*quantize*](#_quantize). This method is useful for interaction, say to determine the value in the domain that corresponds to the pixel location under the mouse.

```js
var s = quantize().domain([10, 100]).range([1, 2, 4]);
s.invertExtent(2); // [40, 70]
```

<a name="quantize_domain" href="#quantize_domain">#</a> <i>quantize</i>.<b>domain</b>([<i>domain</i>])

If *domain* is specified, sets the scale’s domain to the specified two-element array of numbers. If the array contains more than two numbers, only the first and last number are used; if the elements in the given array are not numbers, they will be coerced to numbers. If *domain* is not specified, returns the scale’s current domain.

<a name="quantize_nice" href="#quantize_nice">#</a> <i>quantize</i>.<b>nice</b>()

Equivalent to [*linear*.nice](#linear_nice).

<a name="quantize_ticks" href="#quantize_ticks">#</a> <i>quantize</i>.<b>ticks</b>([<i>count</i>])

Equivalent to [*linear*.ticks](#linear_ticks).

<a name="quantize_tickFormat" href="#quantize_tickFormat">#</a> <i>quantize</i>.<b>tickFormat</b>([<i>count</i>[, <i>specifier</i>]])

Equivalent to [*linear*.tickFormat](#linear_tickFormat).

<a name="quantize_range" href="#quantize_range">#</a> <i>quantize</i>.<b>range</b>([<i>range</i>])

If *range* is specified, sets the scale’s range to the specified array of values. The array may contain any number of discrete values. The elements in the given array need not be numbers; any value or type will work. If *range* is not specified, returns the scale’s current range.

<a name="quantize_copy" href="#quantize_copy">#</a> <i>quantize</i>.<b>copy</b>()

Returns an exact copy of this scale. Changes to this scale will not affect the returned scale, and vice versa.

### Quantile Scales

Quantile scales map an input domain to a discrete range. Although the domain is continuous and the scale will accept any reasonable input value, the domain is specified as a discrete set of values. The number of values in (the cardinality of) the output range determines the number of quantiles that will be computed from the domain. To compute the quantiles, the domain is sorted, and treated as a [population of discrete values](https://en.wikipedia.org/wiki/Quantile#Quantiles_of_a_population). The domain is typically a dimension of the data that you want to visualize, such as the daily change of the stock market. The range is typically a dimension of the desired output visualization, such as a diverging color scale. See [bl.ocks.org/8ca036b3505121279daf](http://bl.ocks.org/mbostock/8ca036b3505121279daf) for an example.

<a name="quantile" href="#quantile">#</a> <b>quantile</b>()

Constructs a new quantile scale with an empty [domain](#quantile_domain) and an empty [range](#quantile_range). The quantile scale is invalid until both a domain and range are specified.

<a name="_quantile" href="#_quantile">#</a> <i>quantile</i>(<i>x</i>)

Given a value *x* in the input [domain](#quantile_domain), returns the corresponding value *y* in the output [range](#quantile_range).

<a name="quantile_invertExtent" href="#quantile_invertExtent">#</a> <i>quantile</i>.<b>invertExtent</b>(<i>y</i>)

Returns the extent of values in the [domain](#quantile_domain) [<i>x0</i>, <i>x1</i>] for the corresponding value in the [range](#quantile_range) *y*: the inverse of [*quantile*](#_quantile). This method is useful for interaction, say to determine the value in the domain that corresponds to the pixel location under the mouse.

<a name="quantile_domain" href="#quantile_domain">#</a> <i>quantile</i>.<b>domain</b>([<i>domain</i>])

If *domain* is specified, sets the domain of the quantile scale to the specified set of discrete numeric values. The array must not be empty, and must contain at least one numeric value; NaN, null and undefined values are ignored and not considered part of the sample population. If the elements in the given array are not numbers, they will be coerced to numbers. A copy of the input array is sorted and stored internally. If *domain* is not specified, returns the scale’s current domain.

<a name="quantile_range" href="#quantile_range">#</a> <i>quantile</i>.<b>range</b>([<i>range</i>])

If *range* is specified, sets the discrete values in the range. The array must not be empty, and may contain any type of value. The number of values in (the cardinality, or length, of) the *range* array determines the number of quantiles that are computed. For example, to compute quartiles, *range* must be an array of four elements such as [0, 1, 2, 3]. If *range* is not specified, returns the current range.

<a name="quantile_quantiles" href="#quantile_quantiles">#</a> <i>quantile</i>.<b>quantiles</b>()

Returns the quantile thresholds. If the [range](#quantile_range) contains *n* discrete values, the returned array will contain *n* - 1 thresholds. Values less than the first threshold are considered in the first quantile; values greater than or equal to the first threshold but less than the second threshold are in the second quantile, and so on. Internally, the thresholds array is used with [bisect](https://github.com/d3/d3-array#bisect) to find the output quantile associated with the given input value.

<a name="quantile_copy" href="#quantile_copy">#</a> <i>quantile</i>.<b>copy</b>()

Returns an exact copy of this scale. Changes to this scale will not affect the returned scale, and vice versa.

### Threshold Scales

Threshold scales are similar to [quantize scales](#quantize-scales), except they allow you to map arbitrary subsets of the domain to discrete values in the range. The input domain is still continuous, and divided into slices based on a set of threshold values. The domain is typically a dimension of the data that you want to visualize, such as the height of students in meters in a sample population. The range is typically a dimension of the desired output visualization, such as a set of colors. See [bl.ocks.org/3306362](http://bl.ocks.org/mbostock/3306362) for an example.

<a name="threshold" href="#threshold">#</a> <b>threshold</b>()

Constructs a new threshold scale with the default [domain](#threshold_domain) [0.5] and the default [range](#threshold_range) [0, 1]. Thus, the default threshold scale is equivalent to the [Math.round](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Math/round) function for numbers; for example threshold(0.49) returns 0, and threshold(0.51) returns 1.

<a name="_threshold" href="#_threshold">#</a> <i>threshold</i>(<i>x</i>)

Given a value *x* in the input [domain](#threshold_domain), returns the corresponding value *y* in the output [range](#threshold_range). For example:

```js
var s = threshold().domain([0, 1]).range(["a", "b", "c"]);
s(-1);   // "a"
s(0);    // "b"
s(0.5);  // "b"
s(1);    // "c"
s(1000); // "c"
```

<a name="threshold_invertExtent" href="#threshold_invertExtent">#</a> <i>threshold</i>.<b>invertExtent</b>(<i>y</i>)

Returns the extent of values in the [domain](#threshold_domain) [<i>x0</i>, <i>x1</i>] for the corresponding value in the [range](#threshold_range) *y*, representing the inverse mapping from range to domain. This method is useful for interaction, say to determine the value in the domain that corresponds to the pixel location under the mouse. For example:

```js
var s = threshold().domain([0, 1]).range(["a", "b", "c"]);
s.invertExtent("a"); // [undefined, 0]
s.invertExtent("b"); // [0, 1]
s.invertExtent("c"); // [1, undefined]
```

<a name="threshold_domain" href="#threshold_domain">#</a> <i>threshold</i>.<b>domain</b>([<i>domain</i>])

If *domain* is specified, sets the scale’s domain to the specified array of values. The values must be in sorted ascending order, or the behavior of the scale is undefined. The values are typically numbers, but any naturally ordered values (such as strings) will work; a threshold scale can be used to encode any type that is ordered. If the number of values in the scale’s range is N+1, the number of values in the scale’s domain must be N. If there are fewer than N elements in the domain, the additional values in the range are ignored. If there are more than N elements in the domain, the scale may return undefined for some inputs. If *domain* is not specified, returns the scale’s current domain.

<a name="threshold_range" href="#threshold_range">#</a> <i>threshold</i>.<b>range</b>([<i>range</i>])

If *range* is specified, sets the scale’s range to the specified array of values. If the number of values in the scale’s domain is N, the number of values in the scale’s range must be N+1. If there are fewer than N+1 elements in the range, the scale may return undefined for some inputs. If there are more than N+1 elements in the range, the additional values are ignored. The elements in the given array need not be numbers; any value or type will work. If *range* is not specified, returns the scale’s current range.

<a name="threshold_copy" href="#threshold_copy">#</a> <i>threshold</i>.<b>copy</b>()

Returns an exact copy of this scale. Changes to this scale will not affect the returned scale, and vice versa.

### Identity Scales

Identity scales are a special case of [linear scales](#linear-scales) where the [domain](#identity_domain) and [range](#identity_range) are identical; the scale and its invert method are both the identity function. These scales are occasionally useful when working with pixel coordinates, say in conjunction with an axis or brush.

<a name="identity" href="#identity">#</a> <b>identity</b>()

Constructs a new identity scale with the default [domain](#identity_domain) [0, 1] and the default [range](#identity_range) [0, 1]. An identity scale is always equivalent to the identity function.

<a name="_identity" href="#_identity">#</a> <i>identity</i>(<i>x</i>)<br>
<a href="#_identity">#</a> <i>identity</i>.<b>invert</b>(<i>x</i>)

Returns the given value *x*.

<a name="identity_domain" href="#identity_domain">#</a> <i>identity</i>.<b>domain</b>([<i>domain</i>])<br>
<a href="#identity_domain">#</a> <i>identity</i>.<b>range</b>([<i>domain</i>])

If *domain* is specified, sets the scale’s domain and range to the specified array of numbers. The array must contain two or more numbers. If the elements in the given array are not numbers, they will be coerced to numbers. If *domain* is not specified, returns the scale’s current domain (or equivalently, range).

<a name="identity_nice" href="#identity_nice">#</a> <i>identity</i>.<b>nice</b>()

Equivalent to [*linear*.nice](#linear_nice).

<a name="identity_ticks" href="#identity_ticks">#</a> <i>identity</i>.<b>ticks</b>([<i>count</i>])

Equivalent to [*linear*.ticks](#linear_ticks).

<a name="identity_tickFormat" href="#identity_tickFormat">#</a> <i>identity</i>.<b>tickFormat</b>([<i>count</i>[, <i>specifier</i>]])

Equivalent to [*linear*.tickFormat](#linear_tickFormat).

<a name="identity_copy" href="#identity_copy">#</a> <i>identity</i>.<b>copy</b>()

Returns an exact copy of this scale. Changes to this scale will not affect the returned scale, and vice versa.

### Sequential Color Scales

<a name="viridis" href="#viridis">#</a> <b>viridis</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/viridis.png" width="100%" height="40" alt="viridis">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and implementing the “viridis” perceptually-uniform color scheme designed by [van der Walt, Smith and Firing](https://bids.github.io/colormap/) for matplotlib.

<a name="inferno" href="#inferno">#</a> <b>inferno</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/inferno.png" width="100%" height="40" alt="inferno">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and implementing the “inferno” perceptually-uniform color scheme designed by [van der Walt and Smith](https://bids.github.io/colormap/) for matplotlib.

<a name="magma" href="#magma">#</a> <b>magma</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/magma.png" width="100%" height="40" alt="magma">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and implementing the “magma” perceptually-uniform color scheme designed by [van der Walt and Smith](https://bids.github.io/colormap/) for matplotlib.

<a name="plasma" href="#plasma">#</a> <b>plasma</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/plasma.png" width="100%" height="40" alt="plasma">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and implementing the “plasma” perceptually-uniform color scheme designed by [van der Walt and Smith](https://bids.github.io/colormap/) for matplotlib.

<a name="cubehelix" href="#cubehelix">#</a> <b>cubehelix</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/cubehelix.png" width="100%" height="40" alt="cubehelix">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and implementing [Green’s default Cubehelix](https://www.mrao.cam.ac.uk/~dag/CUBEHELIX/) color scheme.

<a name="warm" href="#warm">#</a> <b>warm</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/warm.png" width="100%" height="40" alt="warm">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and approximately implementing a 180° rotation of [Niccoli’s perceptual rainbow](https://mycarta.wordpress.com/2013/02/21/perceptual-rainbow-palette-the-method/) color scheme using the Cubehelix color space.

<a name="cool" href="#cool">#</a> <b>cool</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/cool.png" width="100%" height="40" alt="cool">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] and approximately implementing [Niccoli’s perceptual rainbow](https://mycarta.wordpress.com/2013/02/21/perceptual-rainbow-palette-the-method/) color scheme using the Cubehelix color space.

<a name="rainbow" href="#rainbow">#</a> <b>rainbow</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/rainbow.png" width="100%" height="40" alt="rainbow">

Constructs a new sequential scale with the [domain](#linear_domain) [0, 1] combining the [warm](#warm) scale from [0.0, 0.5] followed by the [cool](#cool) scale from [0.5, 1.0], thus implementing the cyclical [less-angry rainbow](http://bl.ocks.org/mbostock/310c99e53880faec2434) color scheme.

### Ordinal Scales

Unlike [linear](#linear-scales) and other quantitative scales, ordinal scales have a discrete domain and range. For example, an ordinal scale might map a set of named categories to a set of colors, or determine the horizontal positions of columns in a column chart.

<a name="ordinal" href="#ordinal">#</a> <b>ordinal</b>()

Constructs a new ordinal scale with an empty [domain](#ordinal_domain) and an empty [range](#ordinal_range). The ordinal scale is invalid (always returning undefined) until a range is specified.

<a name="_ordinal" href="#_ordinal">#</a> <i>ordinal</i>(<i>x</i>)

Given a value *x* in the input [domain](#ordinal_domain), returns the corresponding value *y* in the output [range](#ordinal_range).

If the given value *x* is not in the scale’s [domain](#ordinal_domain), and the range was specified explicitly (as by [range](#ordinal_range) but not [rangeBands](#ordinal_rangeBands), [rangeRoundBands](#ordinal_rangeRoundBands) or [rangePoints](#ordinal_rangePoints)), then *x* is implicitly added to the domain and the next-available value *y* in the range is assigned to *x*, such that this and subsequent invocations of the scale given the same *x* return the same *y*.

<a name="ordinal_domain" href="#ordinal_domain">#</a> <i>ordinal</i>.<b>domain</b>([<i>domain</i>])

If *domain* is specified, sets the domain of the ordinal scale to the specified array of values. The first element in *domain* will be mapped to the first element in the range, the second domain value to the second range value, and so on. Domain values are stored internally in an associative array as a mapping from value to index; the resulting index is then used to retrieve a value from the range. Thus, an ordinal scale's values must be coercible to a string, and the stringified version of the domain value uniquely identifies the corresponding range value. If *domain* is not specified, this method returns the current domain.

Setting the domain on an ordinal scale is optional. If no domain is set, a [range](#ordinal_range) must be set explicitly. Then, each unique value that is passed to the scale function will be assigned a new value from the range; in other words, the domain will be inferred implicitly from usage. Although domains may thus be constructed implicitly, it is still a good idea to assign the ordinal scale's domain explicitly to ensure deterministic behavior, as inferring the domain from usage will be dependent on ordering.

<a name="ordinal_range" href="#ordinal_range">#</a> <i>ordinal</i>.<b>range</b>([<i>range</i>])

If *range* is specified, sets the range of the ordinal scale to the specified array of values. The first element in the domain will be mapped to the first element in *range*, the second domain value to the second range value, and so on. If there are fewer elements in the range than in the domain, the scale will recycle values from the start of the range. If *range* is not specified, this method returns the current range.

This method is intended for when the set of discrete output values is computed explicitly, such as a set of categorical colors. In other cases, such as determining the layout of an ordinal scatterplot or bar chart, you may find the [rangePoints](#ordinal_rangePoints) or [rangeBands](#ordinal_rangeBands) operators more convenient.

<a name="ordinal_rangePoints" href="#ordinal_rangePoints">#</a> <i>ordinal</i>.<b>rangePoints</b>(<i>interval</i>[, <i>padding</i>])

Sets the range from the specified continuous *interval*. The array *interval* contains two elements representing the minimum and maximum numeric value. This interval is subdivided into *n* evenly-spaced **points**, where *n* is the number of (unique) values in the domain. The first and last point may be offset from the edge of the interval according to the specified *padding*, which defaults to zero. The *padding* is expressed as a multiple of the spacing between points. A reasonable value is 1.0, such that the first and last point will be offset from the minimum and maximum value by half the distance between points.

![rangepoints](https://f.cloud.github.com/assets/230541/538689/46d87118-c193-11e2-83ab-2008df7c36aa.png)

```js
var s = ordinal().domain([1, 2, 3, 4]).rangePoints([0, 100]);
s.range(); // [0, 33.333333333333336, 66.66666666666667, 100]
```

<a name="ordinal_rangeRoundPoints" href="#ordinal_rangeRoundPoints">#</a> <i>ordinal</i>.<b>rangeRoundPoints</b>(<i>interval</i>[, <i>padding</i>])

Like [rangePoints](#ordinal_rangePoints), except guarantees that the range values are integers so as to avoid antialiasing artifacts.

```js
var s = ordinal().domain([1, 2, 3, 4]).rangeRoundPoints([0, 100]);
s.range(); // [1, 34, 67, 100]
```

Note that rounding necessarily introduces additional outer padding which is, on average, proportional to the length of the domain. For example, for a domain of size 50, an additional 25px of outer padding on either side may be required. Modifying the range extent to be closer to a multiple of the domain length may reduce the additional padding.

```js
var s = ordinal().domain(range(50)).rangeRoundPoints([0, 95]);
s.range(); // [23, 24, 25, …, 70, 71, 72]
s.rangeRoundPoints([0, 100]);
s.range(); // [1, 3, 5, …, 95, 97, 98]
```

(Alternatively, you could round the output of the scale manually or apply shape-rendering: crispEdges. However, this will result in irregularly spaced points.)

<a name="ordinal_rangeBands" href="#ordinal_rangeBands">#</a> <i>ordinal</i>.<b>rangeBands</b>(<i>interval</i>[, <i>padding</i>[, <i>outerPadding</i>]])

Sets the range from the specified continuous *interval*. The array *interval* contains two elements representing the minimum and maximum numeric value. This interval is subdivided into *n* evenly-spaced **bands**, where *n* is the number of (unique) values in the domain. The bands may be offset from the edge of the interval and other bands according to the specified *padding*, which defaults to zero. The padding is typically in the range [0, 1] and corresponds to the amount of space in the range interval to allocate to padding. A value of 0.5 means that the band width will be equal to the padding width. The *outerPadding* argument is for the entire group of bands; a value of 0 means there will be padding only between rangeBands.

![rangebands](https://f.cloud.github.com/assets/230541/538688/46c298c0-c193-11e2-9a7e-15d9abcfab9b.png)

```js
var s = ordinal().domain([1, 2, 3]).rangeBands([0, 100]);
s.rangeBand(); // 33.333333333333336
s.range(); // [0, 33.333333333333336, 66.66666666666667]
s.rangeExtent(); // [0, 100]
```

<a name="ordinal_rangeRoundBands" href="#ordinal_rangeRoundBands">#</a> <i>ordinal</i>.<b>rangeRoundBands</b>(<i>interval</i>[, <i>padding</i>[, <i>outerPadding</i>]])

Like [rangeBands](#ordinal_rangeBands), except guarantees that range values and band width are integers so as to avoid antialiasing artifacts.

```js
var s = ordinal().domain([1, 2, 3]).rangeRoundBands([0, 100]);
s.range(); // [1, 34, 67]
s.rangeBand(); // 33
s.rangeExtent(); // [0, 100]
```

Note that rounding necessarily introduces additional outer padding which is, on average, proportional to the length of the domain. For example, for a domain of size 50, an additional 25px of outer padding on either side may be required. Modifying the range extent to be closer to a multiple of the domain length may reduce the additional padding.

```js
var s = ordinal().domain(range(50)).rangeRoundBands([0, 95]);
s.range(); // [23, 24, 25, …, 70, 71, 72]
s.rangeRoundBands([0, 100]);
s.range(); // [0, 2, 4, …, 94, 96, 98]
```

(Alternatively, you could round the output of the scale manually or apply shape-rendering: crispEdges. However, this will result in irregularly spaced and sized bands.)

<a name="ordinal_rangeBand" href="#ordinal_rangeBand">#</a> <i>ordinal</i>.<b>rangeBand</b>()

Returns the band width. When the scale’s range is configured with rangeBands or rangeRoundBands, the scale returns the lower value for the given input. The upper value can then be computed by offsetting by the band width. If the scale’s range is set using range or rangePoints, the band width is zero.

<a name="ordinal_rangeExtent" href="#ordinal_rangeExtent">#</a> <i>ordinal</i>.<b>rangeExtent</b>()

Returns a two-element array representing the extent of the scale's range, i.e., the smallest and largest values.

<a name="ordinal_copy" href="#ordinal_copy">#</a> <i>ordinal</i>.<b>copy</b>()

Returns an exact copy of this ordinal scale. Changes to this scale will not affect the returned scale, and vice versa.

### Categorical Color Scales

<a name="category10" href="#category10">#</a> <b>category10</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/category10.png" width="100%" height="40" alt="category10">

Constructs a new [ordinal scale](#ordinal) with a range of ten categorical colors.

<a name="category20" href="#category20">#</a> <b>category20</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/category20.png" width="100%" height="40" alt="category20">

Constructs a new [ordinal scale](#ordinal) with a range of twenty categorical colors.

<a name="category20b" href="#category20b">#</a> <b>category20b</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/category20b.png" width="100%" height="40" alt="category20b">

Constructs a new [ordinal scale](#ordinal) with a range of twenty categorical colors.

<a name="category20c" href="#category20c">#</a> <b>category20c</b>()

<img src="https://raw.githubusercontent.com/d3/d3-scale/master/img/category20c.png" width="100%" height="40" alt="category20c">

Constructs a new [ordinal scale](#ordinal) with a range of twenty categorical colors. This color scale includes color specifications and designs developed by Cynthia Brewer ([colorbrewer.org](http://colorbrewer.org/)).
