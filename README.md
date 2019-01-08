# Complex period comparisons

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/period.svg?style=flat-square)](https://packagist.org/packages/spatie/period)
[![Build Status](https://img.shields.io/travis/spatie/period/master.svg?style=flat-square)](https://travis-ci.org/spatie/period)
[![Quality Score](https://img.shields.io/scrutinizer/g/spatie/period.svg?style=flat-square)](https://scrutinizer-ci.com/g/spatie/period)
[![StyleCI](https://github.styleci.io/repos/159155863/shield?branch=master)](https://github.styleci.io/repos/159155863)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/period.svg?style=flat-square)](https://packagist.org/packages/spatie/period)

This package adds support for comparing multiple dates with each other.
You can calculate the overlaps and differences between n-amount of periods,
as well as some more basic comparisons between two periods.

Periods can be constructed from any type of `DateTime` implementation, 
making this package compatible with custom `DateTime` implementations like Carbon.

Periods are always immutable, there's never the worry about your input dates being changed. 

This package is still a work in progress.

## Installation

You can install the package via composer:

```bash
composer require spatie/period
```

## Usage

**Overlaps with any other period**: 
this method returns a `PeriodCollection` multiple `Period` objects representing the overlaps.

```php
/*
 * A       [========]
 * B                    [==]
 * C                            [=====]
 * CURRENT        [===============]
 *
 * OVERLAP        [=]   [==]    [=]
 */
 
$a = Period::make('2018-01-01', '2018-01-31');
$b = Period::make('2018-02-10', '2018-02-20');
$c = Period::make('2018-03-01', '2018-03-31');

$current = Period::make('2018-01-20', '2018-03-10');

$overlaps = $current->overlap($a, $b, $c); 
```

**Overlap with all periods**: 
this method only returns one period where all periods overlap.

```php
/*
 * A              [============]
 * B                   [==]
 * C                  [=======]
 *
 * OVERLAP             [==]
 */

$a = Period::make('2018-01-01', '2018-01-31');
$b = Period::make('2018-01-10', '2018-01-15');
$c = Period::make('2018-01-10', '2018-01-31');

$overlap = $a->overlapAll($b, $c);
```

**Diff between multiple periods**: 
this method returns a `PeriodCollection` multiple `Period` objects 
representing the diffs between several periods and one.

```php
/*
 * A                   [====]
 * B                               [========]
 * C         [=====]
 * CURRENT      [========================]
 *
 * DIFF             [=]      [====]
 */

$a = Period::make('2018-01-05', '2018-01-10');
$b = Period::make('2018-01-15', '2018-03-01');
$c = Period::make('2017-01-01', '2018-01-02');

$current = Period::make('2018-01-01', '2018-01-31');

$diff = $current->diff($a, $b, $c);
```

**Overlaps with**: this method returns a boolean indicating of two periods overlap or not.

```php
/*
 * A              [============]
 * B                   [===========]
 */

$a = Period::make('2018-01-01', '2018-01-31');
$b = Period::make('2018-01-10', '2018-02-15');

$overlap = $a->overlapsWith($b); // true
```

**Touches with**: this method determines if two periods touch each other.

```php
/*
 * A              [========]
 * B                        [===========]
 */

$a = Period::make('2018-01-01', '2018-01-31');
$b = Period::make('2018-02-01', '2018-02-15');

$overlap = $a->touchesWith($b); // true
```

**Gap**: returns the gap between two periods. 
If no gap exists, `null` is returned. 

```php
/*
 * A              [========]
 * B                           [===========]
 */

$a = Period::make('2018-01-01', '2018-01-31');
$b = Period::make('2018-02-05', '2018-02-15');

$overlap = $a->gap($b); // Period('2018-02-01', '2018-02-04')
```

**Boundaries of a collection**: get one period representing the boundaries of a collection.

```php
/*
 * A                   [====]
 * B                               [========]
 * C           [=====]
 * D                                             [====]
 *
 * BOUNDARIES  [======================================]
 */
 
$collection = new PeriodCollection(
    Period::make('2018-01-01', '2018-01-05'),
    Period::make('2018-01-10', '2018-01-15'),
    Period::make('2018-01-20', '2018-01-25'),
    Period::make('2018-01-30', '2018-01-31')
);

$boundaries = $collection->boundaries();
```

**Gaps of a collection**: get all the gaps of a collection.

```php
/*
 * A                   [====]
 * B                               [========]
 * C         [=====]
 * D                                             [====]
 *
 * GAPS             [=]      [====]          [==]
 */

$collection = new PeriodCollection(
    Period::make('2018-01-01', '2018-01-05'),
    Period::make('2018-01-10', '2018-01-15'),
    Period::make('2018-01-20', '2018-01-25'),
    Period::make('2018-01-30', '2018-01-31')
);

$gaps = $collection->gaps();
```

**Overlap multiple collections**: returns the overlap between collections. 
This means an AND operation between collections, and an OR operation within the same collection.

```php
/*
 * A            [=====]      [===========]
 * B            [=================]
 * C                [====================]
 *
 * OVERLAP          [=]      [====]
 */

$a = new PeriodCollection(
    Period::make('2018-01-01', '2018-01-07'),
    Period::make('2018-01-15', '2018-01-25')
);

$b = new PeriodCollection(
    Period::make('2018-01-01', '2018-01-20')
);

$c = new PeriodCollection(
    Period::make('2018-01-06', '2018-01-25')
);

$overlap = $a->overlap($b, $c);
```

### Working with `PeriodCollection`

Period collections are constructed from several periods:

```php
$collection = new PeriodCollection(
    Period::make('2018-01-01', '2018-01-02'),
    // …
);
```

They may be looped over directly and its contents will be recognised by your IDE:

```php
$collection = new PeriodCollection(/* … */);

foreach ($collection as $period) {
    $period->…
}
```

You may destruct them:

```php
[$firstPeriod, $secondPeriod, $thirdPeriod] = $collection;
```

And finally construct one collection from another:

```php
$newCollection = new PeriodCollection(...$otherCollection);
```

### Boundaries

By default, period comparisons are done with included boundaries. 
This means that these two periods overlap:

```php
$a = Period::make('2018-01-01', '2018-02-01');
$b = Period::make('2018-02-01', '2018-02-28');

$a->overlapsWith($b); // true
```

The length of a period will also include both boundaries:

```php
$a = Period::make('2018-01-01', '2018-01-31');

$a->length(); // 31
```

It's possible to override the boundary behaviour:

```php
$a = Period::make('2018-01-01', '2018-02-01', null, Period::EXCLUDE_END);
$b = Period::make('2018-02-01', '2018-02-28', null, Period::EXCLUDE_END);

$a->overlapsWith($b); // false
```

There are four types of boundary exclusion:

```php
Period::EXCLUDE_NONE;
Period::EXCLUDE_START;
Period::EXCLUDE_END;
Period::EXCLUDE_ALL;
```

### Compatibility

You can construct a `Period` from any type of `DateTime` object such as Carbon:

```php
Period::make(Carbon::make('2018-01-01'), Carbon::make('2018-01-02'));
```

Note that as soon as a period is constructed, all further operations on it are immutable.
There's never the danger of changing the input dates.

### Visualizing periods

You can visualize one or more `Period` objects as well as `PeriodCollection`
objects to see how they related to one another:

```php
$visualizer = new Visualizer(["width" => 27]);

$visualizer->visualize([
    "A" => Period::make('2018-01-01', '2018-01-31'),
    "B" => Period::make('2018-02-10', '2018-02-20'),
    "C" => Period::make('2018-03-01', '2018-03-31'),
    "D" => Period::make('2018-01-20', '2018-03-10'),
    "OVERLAP" => new PeriodCollection(
        Period::make('2018-01-20', '2018-01-31'),
        Period::make('2018-02-10', '2018-02-20'),
        Period::make('2018-03-01', '2018-03-10')
    ),
]);
```

And visualize will return the following string:

```
A          [========]
B                      [==]
C                           [========]
D               [==============]
OVERLAP         [===]  [==] [==]
```

The visualizer has a configurable width provided upon creation
which will control the bounds of the displayed periods:

```php
$visualizer = new Visualizer(["width" => 10]);
```

### Testing

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Postcardware

You're free to use this package, but if it makes it to your production environment we highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using.

Our address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

We publish all received postcards [on our company website](https://spatie.be/en/opensource/postcards).

## Credits

- [Brent Roose](https://github.com/brendt)
- [All Contributors](../../contributors)

## Support us

Spatie is a webdesign agency based in Antwerp, Belgium. You'll find an overview of all our open source projects [on our website](https://spatie.be/opensource).

Does your business depend on our contributions? Reach out and support us on [Patreon](https://www.patreon.com/spatie). 
All pledges will be dedicated to allocating workforce on maintenance and new awesome stuff.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
