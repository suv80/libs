###对日期和时间进行加法运算
**DateTime::add**

**date_add**

面向对象风格

```
<?php
$date = new DateTime('2000-01-01');
$date->add(new DateInterval('P10D'));
echo $date->format('Y-m-d') . "\n";
?>
```

过程化风格

```
<?php
$date = date_create('2000-01-01');
date_add($date, date_interval_create_from_date_string('10 days'));
echo date_format($date, 'Y-m-d');
?>
```

Output:

`
2000-01-11
`

Example:

```
<?php
$date = new DateTime('2000-01-01');
$date->add(new DateInterval('PT10H30S'));
echo $date->format('Y-m-d H:i:s') . "\n";

$date = new DateTime('2000-01-01');
$date->add(new DateInterval('P7Y5M4DT4H3M2S'));
echo $date->format('Y-m-d H:i:s') . "\n";
?>
```

Output：

`
2000-01-01 10:00:30
2007-06-05 04:03:02
`

Example:

```
<?php
$date = new DateTime('2000-12-31');
$interval = new DateInterval('P1M');

$date->add($interval);
echo $date->format('Y-m-d') . "\n";

$date->add($interval);
echo $date->format('Y-m-d') . "\n";
?>
```

以上例程会输出：

`
2001-01-31
2001-03-03
`

###对日期和时间进行减法运算
**DateTime::sub**

**date_sub**

面向对象风格

```
<?php
$date = new DateTime('2000-01-20');
$date->sub(new DateInterval('P10D'));
echo $date->format('Y-m-d') . "\n";
?>
```

过程化风格

```
<?php
$date = date_create('2000-01-20');
date_sub($date, date_interval_create_from_date_string('10 days'));
echo date_format($date, 'Y-m-d');
?>
```

以上例程会输出：
`
2000-01-10
`
Example:

```
<?php
$date = new DateTime('2000-01-20');
$date->sub(new DateInterval('PT10H30S'));
echo $date->format('Y-m-d H:i:s') . "\n";

$date = new DateTime('2000-01-20');
$date->sub(new DateInterval('P7Y5M4DT4H3M2S'));
echo $date->format('Y-m-d H:i:s') . "\n";
?>
```

以上例程会输出：
`
2000-01-19 13:59:30
1992-08-15 19:56:58
`

Example #3:

```
<?php
$date = new DateTime('2001-04-30');
$interval = new DateInterval('P1M');

$date->sub($interval);
echo $date->format('Y-m-d') . "\n";

$date->sub($interval);
echo $date->format('Y-m-d') . "\n";
?>
```

以上例程会输出：
`
2001-03-30
2001-03-02
`
