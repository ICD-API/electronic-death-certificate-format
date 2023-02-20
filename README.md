
# Death Certificate Exchange Format

The main goal of this format is creation of a standard electronic representation for the [International form of medical certificate of cause of death](https://icdcdn.who.int/icd11referenceguide/en/html/index.html#international-form-of-medical-certificate-of-cause-of-death)


The **death certificate** is defined as a *JSON* format. 

[The json schema for a single death certificate can be found here](Annex_ElectronicDeathCertificateSchema.json) 

And a sample file that illustrates the structure of a file that contains an array of certificates is available [here](Annex_ElectronicDeathCertificateSchema.json).


## Overview

In this document each field of the certificate is listed together with its description data type and allowed values. For some fields which can contain only specific 
values we are specifying the mapping, where in the certificate the numeric need to be used. 
> i.e. when the possible values can be “yes”, “no” and “unknown” 
we have the mapping:
> - _no_ -> 0, 
> - _yes_ -> 1, 
> - _unknown_ -> 9. 

We are using `\` to identify nested structures.
> For example `Part2` has nested `Text` which we identify as `Part2\Text`
``` json
"Part2": {
    "Text": "text",
    "Code": "text",
    "LinearizationURI": "text",
    "FoundationURI": "text"
}
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

### Certificate fields

Those fields are used to identify the certificates and other general information such as the ICD version that is used if the certificate contains coded data.

| Attribute |  Type | Description |
| --- | --- | --- | --- |
| `CertificateKey` |  `string` | Can be used to identify the certificate. |
| `Issuer` |  `string` | Can be used to identify the issuer. |
| `Comments` |  `string` | Field that can be filled with free text related to the certificate data or about the computation result. |
| `FreeText` | `string` | Additional field that can be filled with free text related to the certificate data or about the computation result. |
| `ICDVersion`  | `string` | If the certificate contains coded data, this field specifies the ICD version used for the coding of the certificate. e.g.`ICD10` or `ICD11`|
| `ICDMinorVersion`  | `string` | If the certificate contains coded data, this field specifies the ICD minor version used for the coding of the certificate associated to the ICD version.|
| `AdministrativeData` |  `structure` |  Administrative data structure with nested fields below. |
| `AdministrativeData\DateBirth` | `date` ||
| `AdministrativeData\DateDeath` | `date` ||
| `AdministrativeData\Sex` | `integer` | |
| `AdministrativeData\EstimatedAge` | `integer` |  Estimated age if `DateBirth` and `DateDeath` are missing. |

> `Sex` mapping values:  
> - 0 <- “Male”,  
> - 1 <- “Female”,  
> - 9 <- ”Unknown”.

| Attribute |  Type | Description |
| --- | :- | --- | :- | --- |
| `Part1` | `array[structure]` |  List of _cause of death_ structures. Each element can be seen as a condition line of the causality in the certificate. Nested fields allowed were presented before `Text`, `Code`, `LinearizationURI`, `FoundationURI` and `Interval`. |
| `Part2` | `structure`  | _Cause of death_ structure. Each element can be seen as a condition line of the causality in the certificate. Nested fields allowed were presented before `Text`, `Code`, `LinearizationURI`, `FoundationURI` and `Interval`. |


#### Cause of death structure
>> Nested structure used in Part1 and Part2



>> | Attribute | Type | Description |
| --- | --- | --- |
| `Text` | `string` | Textual description or condition choosen by the physician. |
| `Code` | `string` | Classification codes comma separated. (For ICD 11 its allowed to use post coordination, i.e. “Stem A & Ext 1 / Stem B”.) |
| `LinearizationURI` | `string` | Only used for the ICD-11 Linearization URI with possible post coordination (Stem URI A & Ext URI 1 / Stem URI B). For multiple URI’s use comma to separated entities. |
| `FoundationURI` | `string` | Only used for the ICD-11 Foundation URI when the Linearization URI are not sufficient to archive the level of detail needed and with possible post coordination (Stem URI A & Ext URI 1 / Stem URI B). For multiple URI’s use comma to separated entities. |
| | | _Code and URI fields above are used if the certificate contains coded data_ | 
| `Interval` | `string` | Time interval from onset to death. |


| Attribute |  Type | Description |
| --- | :- | --- | :- | --- |
| `Surgery` | `structure`  | Used when surgery was performed. Fill the nested fields. |
| `Surgery\WasPerformed` | `integer` | “Was surgery performed within the last 4 weeks?” |
| `Surgery\Date` |  `boolean`  | Is surgery was performed specify date. |
| `Surgery\Reason` | `string`  | If _yes_, specify reason for surgery (disease or condition). |

> `Surgery\WasPerformed` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.


The following fields are a nested structure used to state condition line of the certificate. They are used by fields as `Part1` and `Part2` 


| Attribute | Required | Type | Used by SMoL computation | Description |
| --- | :-: | --- | :-: | --- |
| `Autopsy` | _optional_ | `structure` | _false_ | If autopsy was requested, fill the nested fields.  |
| `Autopsy\WasRequested` | _optional_ | `integer` | _false_ | “Was an autopsy requested?”. |
| `Autopsy\Findings` | _optional_ | `integer` | _false_ | “If yes were the findings used in the certification?”. |

> `Surgery\WasRequested` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

> `Surgery\Findings` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

| Attribute | Type |  Description |
| --- | :-: | --- | :-: | --- |
| `MannerOfDeath` | `structure` |  Fill the nested fields. |
| `MannerOfDeath\MannerOfDeath` | `integer` |   |
| `MannerOfDeath\DateOfExternalCauseOrPoisoning`  | `date` |   |
| `MannerOfDeath\DescriptionExternalCause` |  `string` |  |
| `MannerOfDeath\PlaceOfOccuranceExternalCause`  | `integer` |  |

> `MannerOfDeath\MannerOfDeath` mapping values:  
> - 0 <- “Disease”,
> - 1 <- “Assault”,
> - 2 <- “Could not be determined”,
> - 3 <- “Accident”,
> - 3 <- “Legal intervention”,
> - 4 <- “Pending investigation”,
> - 5 <- “Intentional self harm”,
> - 6 <- “War”,
> - 9 <- “Unknown”.

> `MannerOfDeath\PlaceOfOccuranceExternalCause` mapping values:  
> - 0 <- “At home”,
> - 1 <- “Residential institution”,
> - 2 <- “School”,
> - 3 <- “other institution”,
> - 4 <- “public administrative area”,
> - 5 <- “Sports and athletics area”,
> - 6 <- “Street and highway”,
> - 7 <- “Trade and service area”,
> - 8 <- “Industrial and construction area”,
> - 9 <- “Farm”,
> - 10 <- “Other place”,
> - 99 <- “Unknown”.

| Attribute |  Type |  Description |
| --- | :-: | --- | :-: | --- |
| `FetalOrInfantDeath` |  `structure` |   |
| `FetalOrInfantDeath\MultiplePregnancy` |  `integer` |   |
| `FetalOrInfantDeath\Stillborn` |  `integer` |   |
| `FetalOrInfantDeath\DeathWithin24h` |  `integer` |   |
| `FetalOrInfantDeath\BirthHeight` |  `integer` |   |
| `FetalOrInfantDeath\PregnancyWeeks` |  `integer` |   |
| `FetalOrInfantDeath\AgeMother` |  `integer` |   |
| `FetalOrInfantDeath\PerinatalDescription` |  `string` |   |

> `FetalOrInfantDeath\MultiplePregnancy` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

> `FetalOrInfantDeath\Stillborn` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

| Attribute |  Type |  Description |
| --- | :-: | --- | :-: | --- |
| `MaternalDeath` | `integer` |   |
| `MaternalDeath\WasPregnant` |  `integer` |   |
| `MaternalDeath\TimeFromPregnancy` |  `integer` |   |
| `MaternalDeath\PregnancyContribute` |  `integer` |   |

> `MaternalDeath\WasPregnant` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

> `MaternalDeath\TimeFromPregnancy` mapping values:  
> - 0 <- “At time of death”,
> - 1 <- “Within 42 days before the death”,
> - 2 <- “Between 43 days up to 1 year before death”,
> - 9 <- “Unknown”.

> `MaternalDeath\PregnancyContribute` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.


## Array of certificates
While we have seen the format of the death certificate, the file with certificates need and array of certificates, which in json is created by using square brackets `[`certificate `,` certificate `,`...`,`certificate `]` 

## Example

Two examples of certificates for ICD10 and ICD11.

#### ICD-10 example

```json
[
    {
        "CertificateKey": "0",
        "Issuer": "",
        "Comments": "",
        "FreeText": "",
        "ICDVersion": "ICD10",
        "ICDMinorVersion": "2016",
        "UCStated": {
            "Text": "Sepsis, unspecified",
            "Code": "A419",
        },
        "UCComputed": {
            "RuleEngine": "",
            "Reject": false,
            "Validate": true,
            "UCCode": "A419",
            "UCText": "Sepsis, unspecified",
        },
        "AdministrativeData": {
            "Sex": 0,
            "DateDeath": "2020-01-01",
            "DateBirth": "2000-01-01",
            "EstimatedAge": 20
        },
        "Part1": [
            {
                "Text": "Other specified general symptoms and signs",
                "Code": "R68.8"
            },
            {
                "Text": "Sepsis, unspecified",
                "Code": "A41.9"
            },
            {
                "Text": "Other and unspecified abdominal pain",
                "Code": "R10.4"
            }
        ],
        "Part2": {},
        "Surgery": {
            "WasPerformed": 1,
            "Reason": "Textual reason",
	    "Date": "2019-05-25"
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

```json
[
    {
        "CertificateKey": "13",
        "Issuer": null,
        "Comments": "Seq: 0",
        "ICDVersion": "ICD11",
        "ICDMinorVersion": "2020",
,
        "AdministrativeData": {
            "Sex": 0,
            "DateDeath": "2020-01-01",
            "DateBirth": "2000-01-01",
            "EstimatedAge": 20
        },
        "Part1": [
            {
                "Text": "",
                "Code": "BA41.Z",
                "LinearizationURI": "http://id.who.int/icd/entity/1334938734/mms/unspecified",
                "Interval": ""
            },
            {
                "Text": "",
                "Code": "5A14",
                "LinearizationURI": "http://id.who.int/icd/entity/1697306310",
                "Interval": ""
            },
            {
                "Text": "",
                "Code": "BA00.Z",
                "LinearizationsURI": "http://id.who.int/icd/entity/761947693/mms/unspecified",
                "Interval": ""
            }
        ],
        "Part2": {
            "Text": "",
            "Code": "",
            "LinearizationsURI": ""
        },
        "Surgery": {
            "WasPerformed": 1,
            "Reason": "Textual reason",
            "Date": "2019-05-25"
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

