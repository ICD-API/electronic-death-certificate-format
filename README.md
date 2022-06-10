
# Death Certificate exchange description

The main goal of this format is creation of a standard electronic representation for the [International form of medical certificate of cause of death](https://icdcdn.who.int/icd11referenceguide/en/html/index.html#international-form-of-medical-certificate-of-cause-of-death)

The proposed **death certificate** is defined as a *json* format. Below we are presenting the structure of the single death certificate defined in its json schema 
(*Annex_UCODToolExchangeSchema.json*) while the file expects an array of certificates and presented as a complete example in the example file (*Annex_UCODToolExchange.json*).
The certificate fields structure follows the example of electronic format of the death certificate.
Following we presenting a logical subdivision of the certificate, describe each field of the certificate and show some examples.

## Logical subdivision

The certificate can be divided in 3 main sections:
1.	One for information’s related to the certificate and provenience, alreasy stated Underlying Cause of Death and version of coding.
2.	One related to the cause of death filled by physicians and codded, which are the information’s that are used by the system to compute the UCOD,
3.	The last one that contains the system computed output, which are the information’s that are computed by the system.

## Description

In this section we are presenting each field of the certificate, each data type allowed and format. For some fields which can contain only specific 
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

### Data type

The data type used are:

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

### Certificate related

Those fields are used to identify the certificates, those informations are not used for the decision of the UCOD, but some as `CodingVerion` 
can be used by the system to choose the classification revision and version. All the fields are set in the first level of the certificate.

| Attribute | Required | Type | Description |
| --- | --- | --- | --- |
| `CertificateKey` | _optional_ | `string` | Can be used to identify the certificate. |
| `Issuer` | _optional_ | `string` | Can be used to identify the issuer. |
| `Comments` | _optional_ | `string` | Field that can be filled with free text related to the certificate data or about the computation result. |
| `FreeText` | _optional_ | `string` | Additional field that can be filled with free text related to the certificate data or about the computation result. |
| `ICDVersion` | _required_ | `string` | Specify the ICD revision used for the coding of the certificate. The system currently implemented three classifications: `ICD10`, `ICD11` and `SMoL`. |
| `ICDMinorVersion` | _required_ | `string` | Specify the ICD minor version used for the coding of the certificate associated to the ICD version. |
| `UCStated` | _optional_ | `structure` | If the certificate has a stated UCOD, use the sub-structure fields `Text`, `Code`, `LinearizationURI`, `FoundationURI`. |

The following fields are a nested structure used to state condition line of the certificate. They are used by fields as `Part1`, `Part2`, 
`UCStated`, `UCComputed\UC` and `UCComputed\NewCodes`. 

| Attribute | Type | Description |
| --- | --- | --- |
| `Text` | `string` | Textual description or condition choosen by the physician. |
| `Code` | `string` | Classification codes comma separated. (For ICD 11 its allowed to use post coordination, i.e. “Stem A & Ext 1 / Stem B”.) |
| `LinearizationURI` | `string` | Only used for the ICD-11 Linearization URI with possible post coordination (Stem URI A & Ext URI 1 / Stem URI B). For multiple URI’s use comma to separated entities. |
| `FoundationURI` | `string` | Only used for the ICD-11 Foundation URI when the Linearization URI are not sufficient to archive the level of detail needed and with possible post coordination (Stem URI A & Ext URI 1 / Stem URI B). For multiple URI’s use comma to separated entities. |
| `Interval` | `string` | Time interval from onset to death. |

### Certificate related cause of death

Following we present the certificate associate data.

| Attribute | Required | Type | Used by SMoL computation | Description |
| --- | :-: | --- | :-: | --- |
| `AdministrativeData` | _optional_ | `structure` | _true_ | Administrative data structure with nested fields below. |
| `AdministrativeData\DateBirth` | _optional_ | `date` | _false_ ||
| `AdministrativeData\DateDeath` | _optional_ | `date` | _false_ ||
| `AdministrativeData\Sex` | _required_ | `integer` | _true_ ||
| `AdministrativeData\EstimatedAge` | _optional_ | `integer` | _false_ | Estimated age if `DateBirth` and `DateDeath` are missing. |

> `Sex` mapping values:  
> - 0 <- “Male”,  
> - 1 <- “Female”,  
> - 9 <- ”Unknown”.

| Attribute | Required | Type | Used by SMoL computation | Description |
| --- | :-: | --- | :-: | --- |
| `Part1` | _required_ | `array[structure]` | _true_ | List of condition structures. Each element can be seen as a condition line of the causality in the certificate. Nested fields allowed were presented before `Text`, `Code`, `LinearizationURI`, `FoundationURI` and `Interval`. |
| `Part2` | _required_ | `structure` | _true_ | Condition tructure. Each element can be seen as a condition line of the causality in the certificate. Nested fields allowed were presented before `Text`, `Code`, `LinearizationURI`, `FoundationURI` and `Interval`. |
| `Surgery` | _optional_ | `structure` | _false_ | Used when surgery was performed. Fill the nested fields. |
| `Surgery\WasPerformed` | _optional_ | `integer` | _false_ | “Was surgery performed within the last 4 weeks?” |
| `Surgery\Date` | _optional_ | `boolean` | _false_ | Is surgery was performed specify date. |
| `Surgery\Reason` | _optional_ | `string` | _false_ | If _yes_, specify reason for surgery (disease or condition). |

> `Surgery\WasPerformed` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

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

| Attribute | Required | Type | Used by SMoL computation | Description |
| --- | :-: | --- | :-: | --- |
| `MannerOfDeath` | _optional_ | `structure` | _false_ | Fill the nested fields. |
| `MannerOfDeath\MannerOfDeath` | _optional_ | `integer` | _false_ |  |
| `MannerOfDeath\DateOfExternalCauseOrPoisoning` | _optional_ | `date` | _false_ |  |
| `MannerOfDeath\DescriptionExternalCause` | _optional_ | `string` | _false_ |  |
| `MannerOfDeath\PlaceOfOccuranceExternalCause` | _optional_ | `integer` | _false_ |  |

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

| Attribute | Required | Type | Used by SMoL computation | Description |
| --- | :-: | --- | :-: | --- |
| `FetalOrInfantDeath` | _optional_ | `structure` | _false_ |  |
| `FetalOrInfantDeath\MultiplePregnancy` | _optional_ | `integer` | _false_ |  |
| `FetalOrInfantDeath\Stillborn` | _optional_ | `integer` | _false_ |  |
| `FetalOrInfantDeath\DeathWithin24h` | _optional_ | `integer` | _false_ |  |
| `FetalOrInfantDeath\BirthHeight` | _optional_ | `integer` | _false_ |  |
| `FetalOrInfantDeath\PregnancyWeeks` | _optional_ | `integer` | _false_ |  |
| `FetalOrInfantDeath\AgeMother` | _optional_ | `integer` | _false_ |  |
| `FetalOrInfantDeath\PerinatalDescription` | _optional_ | `string` | _false_ |  |

> `FetalOrInfantDeath\MultiplePregnancy` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

> `FetalOrInfantDeath\Stillborn` mapping values:  
> - 0 <- “No”,  
> - 1 <- “Yes”,  
> - 9 <- ”Unknown”.

| Attribute | Required | Type | Used by SMoL computation | Description |
| --- | :-: | --- | :-: | --- |
| `MaternalDeath` | _optional_ | `integer` | _false_ |  |
| `MaternalDeath\WasPregnant` | _optional_ | `integer` | _false_ |  |
| `MaternalDeath\TimeFromPregnancy` | _optional_ | `integer` | _false_ |  |
| `MaternalDeath\PregnancyContribute` | _optional_ | `integer` | _false_ |  |

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

### Underlying Cause of Death computed values

The `UCComputed` field is used by the system to add the computed results.

| Attribute | Required | Type | Description |
| --- | --- | --- | --- |
| `RuleEngine` | _required_ | `string` | Used to specify the rule engine used to the computation, only in case of automatic systems. |
| `Coding` | _required_ | `string` | Two values possible “Automatic, Manual”. |
| `Reject` | _required_ | `boolean` | `false` if the computation was able to select the underlying cause of death, `true` otherwise. The computation can fail for multiple reasons (codes not found in the specific linearization, implausibility of the coding, errors of the system.). The reason can be identified in the file logger or rule logger fields. |
| `Validate` | _optional_ | `boolean` | `true` if the computation was not rejected and the stated UCOD is the same with the selected UCOD, `false` otherwise. |
| `NewCodes` | _optional_ | `structure` | Similarly to `Part1` and `Part2` allow the fields: `Text`, `Code`, `LinearizationURI`, `FoundationURI`. Here can be found the new codes added by the system computation for the special instructions (__M1__) when the instruction __Code to__ specify a code not present in the certificate. |
| `UC` | _required_ | `structure` | Similarly to `Part1` and `Part2` allow the fields: `Text`, `Code`, `LinearizationURI`, `FoundationURI`. The computed underlying cause of death computed is stored here, the code, associated textual description and linearization URI and foundation URI. |
| `FileLogger` | _optional_ | `string` | Used only in case of errors, specify the error and reason for which the rule engine has failed the computation. |
| `RuleLogger` | _required_ | `string` | List of operation that the rule engine made to decide the UCOD, based on the level of detail the rules used could be specified. |

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
        "UCStated": {
            "Text": "",
            "Code": "5A14/BA41.Z",
            "LinearizationURI": "http://id.who.int/icd/release/11/mms/1697306310 / http://id.who.int/icd/release/11/mms/1334938734/unspecified"
        },
        "UCComputed": {
            "RuleEngine": "",
            "Reject": false,
            "Validate": true,
            "Coding": "",
            "OutputDescription": "Select potential underlying cause: 5A14\nSelect tentative underlying cause: 5A14\n",
            "LongOutputDescription": "Searching for the starting point SP1-SP5 by evaluating multiple sequences in in Part 1.\n\tLongest sequence: BA41.Z, 5A14\n\tRejected sequences for Diabetes Due To Other Conditions.\n\tSelect potential underlying cause: 5A14\nCheck whether the tentative starting point in the steps SP1-SP5 was selected.\n\tTentaive starting point selected in the Part 1.\n\tSelect tentative underlying cause: 5A14\nCheck for modifications of the starting point, SMoL rules (WIP on Steps M1-M4)\n\tThe tentative underlying cause is the same as the starting point selected in Steps SP1 to SP8. Go to step M4.\n\t",
            "NewCodes": {},
            "UC": {
                "Code": "5A14",
                "Text": "DIABETES MELLITUS",
                "LinearizationURI": "http://id.who.int/icd/release/11/mms/1697306310"
            }
        },
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
                "LinearizationURI": "http://id.who.int/icd/release/11/mms/1334938734/unspecified",
                "Interval": ""
            },
            {
                "Text": "",
                "Code": "5A14",
                "LinearizationURI": "http://id.who.int/icd/release/11/mms/1697306310",
                "Interval": ""
            },
            {
                "Text": "",
                "Code": "BA00.Z",
                "LinearizationsURI": "http://id.who.int/icd/release/11/mms/761947693/unspecified",
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

