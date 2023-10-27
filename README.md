
# Death Certificate Exchange Format

The main goal of this format is creation of a standard electronic representation for the [International form of medical certificate of cause of death](https://icdcdn.who.int/icd11referenceguide/en/html/index.html#international-form-of-medical-certificate-of-cause-of-death)


The **death certificate** is defined as a *JSON* format. 

[The json schema for a single death certificate can be found here](Annex_ElectronicDeathCertificateSchema.json) 


## Overview

In this document each field of the certificate is listed together with its description data type and allowed values. For some fields which can contain only specific 
values we are specifying the mapping, where in the certificate the numeric need to be used. 

i.e. when the possible values can be “yes”, “no” and “unknown” 
we have the mapping:
> - _no_ -> 0, 
> - _yes_ -> 1, 
> - _unknown_ -> 9. 

We are using `\` to identify nested structures.
```
```

### Data types

The following data types are used:

| Type | Description |
| --- | --- |
| `string` | Alphanumeric value insert between quotation marks `"`. |
| `integer` | Numeric field, whole numbers allowed |
| `boolean` | The values allowed are `true` and `false` |
| `structure` | _json_ structure with other fields inside |
| `array[type]` | A list of the specific type, in our format we are using only list of other structures. i.e. array of `[ {"name": "The name"}, {"name": "Other example name name"} ]` |
| `date` | The date field used in the certificate is using the format defined in the [W3C](https://www.w3.org/TR/NOTE-datetime). The date value need to be insert between quotation marks `"`. |
| `duration` | Durations define the amount of intervening time in a time interval used in the certificate for the interval field. The format is defined in the [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). The duration value need to be insert between quotation marks `"`.|

Example of date:

```
    Year:
        YYYY (eg 1997)  
    Year and month:  
        YYYY-MM (eg 1997-07)  
    Complete date:  
        YYYY-MM-DD (eg 1997-07-16)  
    Complete date plus hours and minutes:  
        YYYY-MM-DDThh:mmTZD (eg 1997-07-16T19:20+01:00)  
    Complete date plus hours, minutes and seconds:  
        YYYY-MM-DDThh:mm:ssTZD (eg 1997-07-16T19:20:30+01:00)  
    Complete date plus hours, minutes, seconds and a decimal fraction of a second  
        YYYY-MM-DDThh:mm:ss.sTZD (eg 1997-07-16T19:20:30.45+01:00)
where:
     YYYY = four-digit year
     MM   = two-digit month (01=January, etc.)
     DD   = two-digit day of month (01 through 31)
     hh   = two digits of hour (00 through 23) (am/pm NOT allowed)
     mm   = two digits of minute (00 through 59)
     ss   = two digits of second (00 through 59)
     s    = one or more digits representing a decimal fraction of a second
     TZD  = time zone designator (Z or +hh:mm or -hh:mm)
```

Example of duration:

```
The duration is represented by the format P[n]Y[n]M[n]DT[n]H[n]M[n]S, P[n]W or P<date>T<time>
In these representations, the [n] is replaced by the value for each of the date and time elements that follow the [n]. Leading zeros are not required, but the maximum number of digits for each element should be agreed to by the communicating parties. The capital letters P, Y, M, W, D, T, H, M, and S are designators for each of the date and time elements and are not replaced.
where:
    P is the duration designator (for period) placed at the start of the duration representation.
    Y is the year designator that follows the value for the number of calendar years.
    M is the month designator that follows the value for the number of calendar months.
    W is the week designator that follows the value for the number of weeks.
    D is the day designator that follows the value for the number of calendar days.
    T is the time designator that precedes the time components of the representation.
    H is the hour designator that follows the value for the number of hours.
    M is the minute designator that follows the value for the number of minutes.
    S is the second designator that follows the value for the number of seconds.

For example, "P3Y6M4DT12H30M5S" represents a duration of "three years, six months, four days, twelve hours, thirty minutes, and five seconds".
To resolve ambiguity, "P1M" is a one-month duration and "PT1M" is a one-minute duration.

Practical examples:
    "PT10S" is a ten seconds duration
    "PT10M" is a ten minutes duration
    "PT10H" is a ten hours duration
    "P5D" is a five days duration
    "P2W" is a two weeks duration
    "P10M" is a ten months duration
    "P10Y" is a ten years duration
    "", "P" or "PT" is used for unknown interval.
```

### Certificate Fields


| Attribute |  Type | Description |
| --- | --- | --- | 
| `CertificateKey` |  `string` | Can be used to identify the certificate. |
| `Issuer` |  `string` | Can be used to identify the issuer. |
| `ICDVersion`  | `string` | If the certificate contains coded data, this field specifies the ICD version used for the coding of the certificate. e.g.`ICD10` or `ICD11`|
| `ICDMinorVersion`  | `string` | If the certificate contains coded data, this field specifies the ICD minor version used for the coding of the certificate associated to the ICD version.|
| `AdministrativeData` |  `structure` |  Administrative data structure with nested fields below. |
| `AdministrativeData\DateBirth` | `date` ||
| `AdministrativeData\DateDeath` | `date` ||
| `AdministrativeData\Sex` | `integer` | |
| `AdministrativeData\EstimatedAge` | `duration` |  Estimated age if `DateBirth` and `DateDeath` are missing. |

> `Sex` mapping values:  
> - 1 <- Male
> - 2 <- Female
> - 9 <- Unknown

| Attribute |  Type | Description |
| :- | :- | --- | 
| `Part1` | `[line structure]` |  List of  _line structure_ for each line in Part 1 |
| `Part2` | `line structure`  | _line structure_ having all conditions in Part2  |


#### Single Condition structure

| Attribute | Type | Description |
| :- | :- | --- |
| `Text` | `string` | Textual description or condition choosen by the physician. |
| `Code` | `string` | Classification code. (For ICD 11 its allowed to use post coordination, i.e. “Stem A & Ext 1 / Stem B”.) |
| `LinearizationURI` | `string` | Only used for the ICD-11. Linearization URI can contain post coordination (Stem URI A & Ext URI 1 / Stem URI B). |
| `FoundationURI` | `string` | Only used for the ICD-11 Foundation URI when the Linearization URI are not sufficient to archive the level of detail needed and with possible post coordination (Stem URI A & Ext URI 1 / Stem URI B).  |
| `Interval` | `duration` | Time interval from onset to death. |
||| _The fields above (Code and URIs) are to be used if the certificate contains coded information. In the case for a coded certificate one of the three is sufficient but a certificate could have several filled_ |



#### Line structure

| Attribute | Type | Description |
| --- | --- | --- |
| `Conditions` | `[Single condition structure]` | array of conditions in a single line in Part1 |


| Attribute |  Type | Description |
| --- | :- | --- | 
| `Surgery` | `structure`  | Used when surgery was performed. Fill the nested fields. |
| `Surgery\WasPerformed` | `integer` | “Was surgery performed within the last 4 weeks?” |
| `Surgery\Date` |  `date`  | Is surgery was performed specify date. |
| `Surgery\Reason` | `string`  | If _yes_, specify reason for surgery (disease or condition). |

> `Surgery\WasPerformed` mapping values:  
> - 0 <- No  
> - 1 <- Yes 
> - 9 <- Unknown


The following fields are a nested structure used to state condition line of the certificate. They are used by fields as `Part1` and `Part2` 


| Attribute |  Type |  Description |
| --- | :-: | --- |
| `Autopsy` |  `structure` |  If autopsy was requested, fill the nested fields.  |
| `Autopsy\WasRequested` |  `integer` |  “Was an autopsy requested?”. |
| `Autopsy\Findings` |  `integer` |  “If yes were the findings used in the certification?”. |

 `Autopsy\WasRequested` mapping values:  
> - 0 <- No
> - 1 <- Yes
> - 9 <- Unknown

 `Autopsy\Findings` mapping values:  
> - 0 <- No
> - 1 <- Yes
> - 9 <- Unknown

| Attribute | Type |  Description |
| --- | :-: | --- | 
| `MannerOfDeath` | `structure` |  Fill the nested fields. |
| `MannerOfDeath\MannerOfDeath` | `integer` |   |
| `MannerOfDeath\DateOfExternalCauseOrPoisoning`  | `date` |   |
| `MannerOfDeath\DescriptionExternalCause` |  `string` |  |
| `MannerOfDeath\PlaceOfOccuranceExternalCause`  | `integer` |  |

 `MannerOfDeath\MannerOfDeath` mapping values:  
> - 0 <- Disease
> - 1 <- Accident
> - 2 <- Intentional self harm
> - 3 <- Assault
> - 4 <- Legal intervention
> - 5 <- War
> - 6 <- Could not be determined
> - 7 <- Pending investigation
> - 9 <- Unknown

 `MannerOfDeath\PlaceOfOccuranceExternalCause` mapping values:  
> - 0 <- At home
> - 1 <- Residential institution
> - 2 <- School
> - 3 <- other institution
> - 4 <- public administrative area
> - 5 <- Sports and athletics area
> - 6 <- Street and highway
> - 7 <- Trade and service area
> - 8 <- Industrial and construction area
> - 9 <- Farm
> - 10 <- Other place
> - 99 <- Unknown

| Attribute |  Type |  Description |
| --- | :-: | --- | 
| `FetalOrInfantDeath` |  `structure` |   |
| `FetalOrInfantDeath\MultiplePregnancy` |  `integer` |   |
| `FetalOrInfantDeath\Stillborn` |  `integer` |   |
| `FetalOrInfantDeath\DeathWithin24h` |  `integer` |   |
| `FetalOrInfantDeath\BirthHeight` |  `integer` |   |
| `FetalOrInfantDeath\PregnancyWeeks` |  `integer` |   |
| `FetalOrInfantDeath\AgeMother` |  `integer` |   |
| `FetalOrInfantDeath\PerinatalDescription` |  `string` |   |

 `FetalOrInfantDeath\MultiplePregnancy` mapping values:  
> - 0 <- No
> - 1 <- Yes
> - 9 <- Unknown

 `FetalOrInfantDeath\Stillborn` mapping values:  
> - 0 <- No
> - 1 <- Yes
> - 9 <- Unknown

| Attribute |  Type |  Description |
| --- | :-: | --- | 
| `MaternalDeath` | `structure` |   |
| `MaternalDeath\WasPregnant` |  `integer` |   |
| `MaternalDeath\TimeFromPregnancy` |  `integer` |   |
| `MaternalDeath\PregnancyContribute` |  `integer` |   |

 `MaternalDeath\WasPregnant` mapping values:  
> - 0 <- No
> - 1 <- Yes
> - 9 <- Unknown

 `MaternalDeath\TimeFromPregnancy` mapping values:  
> - 0 <- At time of death
> - 1 <- Within 42 days before the death
> - 2 <- Between 43 days up to 1 year before death
> - 3 <- One year or more before death
> - 9 <- Unknown

 `MaternalDeath\PregnancyContribute` mapping values:  
> - 0 <- No
> - 1 <- Yes
> - 9 <- Unknown


## Array of certificates
While we have seen the format of the death certificate, the file with certificates need and array of certificates, which in json is created by using square brackets `[`certificate `,` certificate `,`...`,`certificate `]` 

## Example

Two examples of certificates for ICD10 and ICD11.

#### ICD-10 example
```
[
    {
        "CertificateKey": "0",
        "Issuer": "",
        "ICDVersion": "ICD10",
        "ICDMinorVersion": "2019",
        "AdministrativeData": {
                "Sex": 1,
                "DateDeath": "2020-01-01",
                "DateBirth": "2000-01-01"
            },
        "Part1": [
            {
                "Conditions": [
                {
                    "Text": "Other specified general symptoms and signs",
                    "Code": "R68.8",
                    "Interval": "P10Y"
                }
                ]
            },
            {
                "Conditions": [
                {
                    "Text": "Sepsis, unspecified",
                    "Code": "A41.9",
                    "Interval": "P10Y"
                }
                ]
            },
            {
                "Conditions": [
                    {
                        "Text": "Other and unspecified abdominal pain",
                        "Code": "R10.4",
                        "Interval": "P10Y"
                    }
                ]
            }
        ],
        "Part2": {
            "Conditions": []
        },
        "Surgery": {
            "WasPerformed": 1,
            "Date": "2019-05-25",
            "Reason": "Textual reason"
        },
        "MannerOfDeath": {
            "MannerOfDeath": 1,
            "DateOfExternalCauseOrPoisoning": "2019-06-27",
            "DescriptionExternalCause": "Textual description",
            "PlaceOfOccuranceExternalCause": 3
        },
        "MaternalDeath": {
            "WasPregnant": 9,
            "TimeFromPregnancy": 9,
            "PregnancyContribute": 9
        }
    }
]
```
#### ICD-11 example
```
[
    {
        "CertificateKey": "13",
        "Issuer": "",
        "ICDVersion": "ICD11",
        "ICDMinorVersion": "2023",
        "AdministrativeData": {
            "Sex": 1,
            "EstimatedAge": "P20Y"
        },
        "Part1": [
        {
            "Conditions": [
                {
                    "Text": "Acute myocardial infarction, unspecified",
                    "Code": "BA41.Z",
                    "LinearizationURI": "http://id.who.int/icd/release/11/mms/1334938734/unspecified",
                    "Interval": "P10Y"
                }
            ]
        },
        {
            "Conditions": [
                {
                    "Text": "Diabetes mellitus, type unspecified",
                    "Code": "5A14",
                    "LinearizationURI": "http://id.who.int/icd/release/11/mms/1697306310",
                    "Interval": "P10Y"
                }
            ]
        },
        {
            "Conditions": [
                {
                    "Text": "Essential hypertension, unspecified",
                    "Code": "BA00.Z",
                    "LinearizationURI": "http://id.who.int/icd/release/11/mms/761947693/unspecified",
                    "Interval": "P10Y"
                }
            ]
        }
        ],
        "Part2": {
            "Conditions": []
        },
        "Surgery": {
            "WasPerformed": 1,
            "Date": "2019-05-25",
            "Reason": "Textual reason"
        },
        "MannerOfDeath": {
            "MannerOfDeath": 1,
            "DateOfExternalCauseOrPoisoning": "2019-06-27",
            "DescriptionExternalCause": "Textual description",
            "PlaceOfOccuranceExternalCause": 3
        },
        "MaternalDeath": {
            "WasPregnant": 9,
            "TimeFromPregnancy": 9,
            "PregnancyContribute": 9
        }
    }
]
```
