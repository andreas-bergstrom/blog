---
title: "Master date and time handling in Python"
date: 2019-03-28T08:38:33+01:00
draft: false
toc: false
images:
tags: 
  - python
---

It might seem like a basic advice, but you will be suprised how much time you can save by simply learning how to properly work with dates. While this is a common task it is often not practiced enough that you really remember all the ins and outs of it. But it is really simple!

When dealing with dates you will want to use the datetime packages, which comes with both date and datetime. The former is for working with dates and the latter for dates and time. There is also the less used time module. We will use datetime for the upcoming examples:

{{< highlight python "linenos=table" >}}
>>> from datetime import datetime
{{< / highlight >}}

When working with dates you will also often use the timedelta module, which expresses the difference between two date, time or datetimes objects:

{{< highlight python "linenos=table" >}}
>>> from datetime import timedelta
{{< / highlight >}}

To get todays date as a datetime-object:

{{< highlight python "linenos=table" >}}
>>> datetime.today()
datetime.datetime(2019, 3, 27, 10, 38, 18, 610489)
{{< / highlight >}}

To create a datetime object you need to pass at least year, month and day:

{{< highlight python "linenos=table" >}}
>>> datetime(year=2019, month=4, day=1)
datetime.datetime(2019, 4, 1, 0, 0)
{{< / highlight >}}

We can get the duration between two dates as a timedelta object by using subtraction:

{{< highlight python "linenos=table" >}}
>>> datetime(year=2019, month=4, day=1) - datetime.today()
datetime.timedelta(days=4, seconds=47625, microseconds=375671)
{{< / highlight >}}

Comparison can be used to determinate if a date is in the past or in the future:

{{< highlight python "linenos=table" >}}
>>> datetime(year=2019, month=4, day=1) > datetime.today()
True
{{< / highlight >}}

By adding a timedelta to a datetime you get a new datetime:

{{< highlight python "linenos=table" >}}
>>> datetime(year=2019, month=1, day=1) + timedelta(days=31)
datetime.datetime(2019, 2, 1, 0, 0)
{{< / highlight >}}

Subtracting a timedelta from another timedelta returns a new timedelta object:

{{< highlight python "linenos=table" >}}
>>> timedelta(weeks=2) - timedelta(days=5)
datetime.timedelta(days=9)
{{< / highlight >}}

Just as with objects of type datetime (and date and time) we can use comparison on timedeltas:

{{< highlight python "linenos=table" >}}
>>> timedelta(days=6) > timedelta(days=3)
True
{{< / highlight >}}

Every datetime object has a handful of attributes that allows you to easily get different parts of the date:

{{< highlight python "linenos=table" >}}
>>> datetime.today().year
2019
>>> datetime.today().month
3
>>> datetime.today().day
27
>>> datetime.today().hour
11
>>> datetime.today().minute
19
>>> datetime.today().second
14
>>> datetime.today().microsecond
346797
{{< / highlight >}}

Since a datetime includes both the date and time, it can easily return either as a new date or time object:

{{< highlight python "linenos=table" >}}
>>> datetime.today().date()
datetime.date(2019, 3, 27)
>>> datetime.today().time()
datetime.time(11, 22, 12, 409267)
{{< / highlight >}}

Converting to and from timestamps:

{{< highlight python "linenos=table" >}}
>>> datetime.today().timestamp()
1553686909.903327
>>> datetime.fromtimestamp(1553686909.903327)
datetime.datetime(2019, 3, 27, 12, 41, 49, 903327)
{{< / highlight >}}

Parsing a string into a date, time or datetime object:

{{< highlight python "linenos=table" >}}
>>> datetime.strptime('2000-01-01', '%Y-%m-%d')
datetime.datetime(2000, 1, 1, 0, 0)
{{< / highlight >}}

Formating a date, time or datetime object into a string:

{{< highlight python "linenos=table" >}}
>>> datetime.now().strftime('%c')
'Wed Mar 27 12:49:48 2019'
>>> datetime.now().strftime('%A %-d %B %Y')
'Wednesday 27 March 2019'
{{< / highlight >}}

And finally a topic that sends chills down the spines of many programmers; timezones. The easiest way to handle those in Python is to use the 3rd party [pytz](https://pypi.org/project/pytz/) package:

{{< highlight python "linenos=table" >}}
>>> from pytz import timezone
{{< / highlight >}}

By default all datetimes in Python is said to be ***naive***, meaning they lack any timezone information. The pytz package can make a datetime object ***aware***, which its data will include the timezone:

{{< highlight python "linenos=table" >}}
>>> datetime.now()
datetime.datetime(2019, 3, 27, 13, 7, 58, 994510)
>>> tz = timezone('Europe/Stockholm')
>>> datetime.now(tz)
datetime.datetime(2019, 3, 27, 13, 7, 55, 45471, tzinfo=<DstTzInfo 'Europe/Stockholm' CET+1:00:00 STD>)
{{< / highlight >}}

It is also possible to localize an existing datetime:

{{< highlight python "linenos=table" >}}
>>> tz.localize(datetime(2011,11,11))
datetime.datetime(2011, 11, 11, 0, 0, tzinfo=<DstTzInfo 'Europe/Stockholm' CET+1:00:00 STD>)
{{< / highlight >}}

Converting from one timezone to another is done through the astimezone() method. Notice the date and time does not change, only the timezone:

{{< highlight python "linenos=table" >}}
>>> dt_now = datetime.now(tz)
>>> dt_now
datetime.datetime(2019, 3, 27, 13, 10, 59, 952962, tzinfo=<DstTzInfo 'Europe/Stockholm' CET+1:00:00 STD>)
>>> dt_now.astimezone(timezone('US/Samoa'))
datetime.datetime(2019, 3, 27, 1, 10, 59, 952962, tzinfo=<DstTzInfo 'US/Samoa' SST-1 day, 13:00:00 STD>)
{{< / highlight >}}

To view all available timezones use pytz.all_timezones:

{{< highlight python "linenos=table" >}}
>>> from pytz import all_timezones 
>>> all_timezones
['Africa/Abidjan', 'Africa/Accra', 'Africa/Addis_Ababa'...
{{< / highlight >}}
