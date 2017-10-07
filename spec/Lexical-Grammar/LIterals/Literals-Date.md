[Lexical Grammar](/Lexical-Grammar) / [Literals](Literals) / Date Literal    
Related: [Nothing Literal](Literals#Nothing-Literal), [Boolean Literal](Literals#Boolean-Literal), [Integer](#Integer-Literal), [Floating Point](Literals-FloatingPoint#Integer-Literal), [String](Literals-String#String-Literal), [Character](Literals-String#Character-Literal), **[Date](Literals-Date#Date-Literal)**

----

## Date Literals

A date literal represents a particular moment in time expressed as a value of the `System.Date` type.

```antlr
DateLiteral  :  '#' WhiteSpace* DateOrTime WhiteSpace* '#'  ;
DateOrTime   :   DateValue WhiteSpace+ TimeValue
               | DateValue
               | TimeValue  ;

DateValue    :   MMDDYYYY  |  YYYYMMDD
  
MMDDYYYY     :   MonthValue '/' DayValue   '/' YearValue
               | MonthValue '-' DayValue   '-' YearValue  ;
YYYYMMDD     :   YearValue  '/' MonthValue '/' DayValue   
               | YearValue  '-' MonthValue '-' DayValue   ;

TimeValue    :   HourValue ':' MinuteValue ( ':' SecondValue )? WhiteSpace* AMPM?
               | HourValue WhiteSpace* AMPM  ;

MonthValue   :   IntLiteral  ;
DayValue     :   IntLiteral  ;
YearValue    :   IntLiteral  ;
HourValue    :   IntLiteral  ;
MinuteValue  :   IntLiteral  ;
SecondValue  :   IntLiteral  ;
       AMPM  :   'AM' | 'PM' ;    
```

The literal may specify both a date and a time, just a date, or just a time.
 * If the date value is omitted, then January 1 of the year 1 in the Gregorian calendar is assumed.
 * If the time value is omitted, then 12:00:00 AM is assumed.

To avoid problems with interpreting the year value in a date value, the year value cannot be two digits. When expressing a date in the first century AD/CE, leading zeros must be specified.

A time value may be specified either using;
 * a 24-hour value or
 * a 12-hour value. 
Time values that omit an `AM` or `PM` are assumed to be 24-hour values.
 * If a time value omits the minutes, the literal `0` is used by default.
 * If a time value omits the seconds, the literal `0` is used by default.
 * If both minutes and second are omitted, then `AM` or `PM` must be specified.
 * If the date value specified is outside the range of the `System.Date` type, a compile-time error occurs.

The following example contains several date literals.

```vb
Dim d As Date

d = # 8/23/1970 3:45:39AM #
d = # 8/23/1970 #              ' Date value: 8/23/1970 12:00:00AM.
d = # 3:45:39AM #              ' Date value: 1/1/1 3:45:39AM.
d = # 3:45:39 #                ' Date value: 1/1/1 3:45:39AM.
d = # 13:45:39 #               ' Date value: 1/1/1 1:45:39PM.
d = # 1AM #                    ' Date value: 1/1/1 1:00:00AM.
d = # 13:45:39PM #             ' This date value is not valid.
```
----

**Prev:** [String Literals](Literals-String)