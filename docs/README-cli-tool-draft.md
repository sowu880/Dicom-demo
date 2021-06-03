# Overview

The Digital Imaging and Communication in Medicine (DICOM) standard has been commonly used for storing, viewing, and transmitting information in medical imaging. A DICOM file not only contains a viewable image but also a header with a large variety of data elements. These meta-data elements include identifiable information about the patient, the study, and the institution. Sharing such sensitive data demands proper protection to ensure data safety and maintain patient privacy.

DICOM Anomymization Tool is an open-source project that helps anonymize metadata in DICOM files. A command-line tool is provided in this project.

# Features
- Support de-identification methods for DICOM metadata including redact, keep, encrypt, cryptoHash, dateShift, perturb, substitute, remove and refreshUID.
- Configuration of the data elements that need to be de-identified.
- Configuration of the de-identification methods for each data element.
- Ability to run the tool on premise to de-identify a dataset locally.

# Quickstarts

## Build the solution
Use the .Net Core 3.1 SDK to build DICOM Anonymization Tool. If you don't have .Net Core 3.1 installed, instructions and download links are available [here](https://dotnet.microsoft.com/download/dotnet/3.1).

## Prepare DICOM Data
You can prepare your own DICOM files as input, or use sample DICOM files in folder $SOURCE\DICOM\samples of the project.

## Anonymize DICOM data using the command line tool

Once you have built the command line tool, you will find executable file Microsoft.Health.Dicom.Anonymizer.CommandLineTool.exe in the $SOURCE\DICOM\src\Microsoft.Health.Dicom.Anonymizer.CommandLineTool\bin\Debug|Release\netcoreapp3.1 folder.

You can use this executable file to anonymize DICOM file.

```
> .\Microsoft.Health.Dicom.Anonymizer.CommandLineTool.exe -i myInputFile -o myOutputFile
```

See the tutorials section for usage details of the command line tool.

# Tutorials
## Use Command Line Tool
The command-line tool can be used to anonymize one DICOM file or a folder containing DICOM files. Here are the parameters that the tool accepts:


| Option | Name | Optionality | Default | Description |
| ----- | ----- | ----- |----- |----- |
| -i | inputFile | Optional | | Input DICOM file. |
| -o | outputFile | Optional | |  Output DICOM file. |
| -c | configFile | Optional |configuration.json | Anonymizer configuration file path. It reads the default file from the current directory. |
| -I | inputFolder | Optional|  | Input folder. |
| -O | outputFolder | Optional |  | Output folder. |
| -v | autoValidate | Optional | true | Auto validate output value when anonymizing. |
| -s | skipFailedItem | Optional | true | Skip failed DICOM files when anonymizing. |
| --validateInput | validateInput | Optional | false | Validate input DICOM file against value multiplicity, value types and format in DICOM specification. |

>Note: To anonymize one DICOM file, inputFile and outputFile are required. To anonymize a DICOM folder, inputFolder and outputFolder are required.

Example usage to anonymize DICOM files in a folder:
```
>.\Microsoft.Health.Dicom.Anonymizer.CommandLineTool.exe convert -I myInputFolder -O myOutputFolder -c myConfigFile
```

## Sample Configuration File
The configuration is specified in JSON format and has three required high-level sections. The first section named _rules_, it specifies de-identifiaction methods for DICOM tag. The second and third sections are _defaultSettings_ and _customizedSettings_ which specify default settings and customized settings for de-identification methods respectively.

|Fields|Description|
|----|----|
|rules|De-ID rules for tags.|
|defaultSettings|Default settings for de-identification functions. Default settings will be used if not specify settings in rules.|
|customizedSettings|Customized settings for de-identification functions.|


DICOM Anonymization tool comes with a sample configuration file to help meet the requirements of HIPAA Safe Harbor Method. DICOM standard also describes attributes within a DICOM dataset that may potentially result in leakage of individually identifiable information according to HIPAA Safe Harbor. Our tool will build in a sample [configuration file]() that covers [application level confidentiality profile attributes](http://dicom.nema.org/medical/dicom/2018e/output/chtml/part15/chapter_E.html) defined in DICOM standard.


## Customize Configuration File

### How to set rules

Users can list de-identification rules for individual DICOM tag ( by tag value or tag name) as well as a set of tags (by masked value or DICOM VR). Ex：
```
{
    "rules": [
            {"tag": "(0010,1010)","method": "perturb"}, 
            {"tag": "(0040,xxxx)",  "method": "redact"},
            {"tag": "PatientID",  "method": "cryptohash"},
            {"tag": "PN", "method": "encrypt"}
    ]
}
```
Parameters in each rules:

|Fields|Description| Valid Value|Required|default value|
|--|-----|-----|--|--|
|tag|Used to define DICOM elements |1. Tag Value, e.g. (0010, 0010) or 0010,0010 or 00100010. <br>2. Tag Name. e.g. PatientName. <br> 3. Masked DICOM Tag. e.g. (0010, xxxx) or (xx10, xx10). <br> 4. DICOM VR. e.g. PN, DA.|True|null| 
|method|De-ID method.| keep, redact, perturb, dateshift, encrypt, cryptohash, substitute, refreshUID, remove.| True|null|
|setting| Setting for de-id methods. Users can add customized settings in the field of "customizedSettings" and specify setting's name here. |valid setting's name |False|Default setting in the field of "defaultSettings"|
|params|parameters override setting for de-id methods.|valid parameters|False|null|

Each DICOM tag can only be de-identified once, if two rules have conflicts on one tag, only the former rule will be applied.

### How to set settings
_defaultSettings_ and _customizedSettings_ are used to config de-identification method. (Detailed parameters are defined in [De-IDentification Methods Section]()). _defaultSettings_ are used when user does not specify settings in rule. As for _customizedSettings_, users need to add the setting with unique name. This setting can be used in "rules" by name.

Here is an example, the first rule will use perturb setting in _defaultSettings_ and the second one will use perturbCustomerSetting in field _cutomizedSettings_.

```
{
    "rules": [
        {"tag": "(0010,0020)","method": "perturb"},
        {"tag": "(0010,1010)","method": "perturb", "setting":"perturbCustomerSetting"}
    ],
    "defaultSettings":[
        {"perturb":{ "span": "1", "roundTo": 2, "rangeType": "Proportional", "NoiseFunction" : "Uniform"}}
    ],
    "cutomizedSettings":[
        {"perturbCustomerSetting":{ "span": "1", "roundTo": 2, "rangeType": "Proportional", "NoiseFunction" : "Uniform"}}
    ]
}
```

## De-Identification Algorithms

### Overview


|De-id Method|Description|Setting Configuration|
|-----|-----|-----|
|keep|Retains the value as is.|No|
|redact|Clean the value.|Yes|
|remove|Remove the element. |No|
|perturb|Perturb the value with random noise addition.|Yes|
|dateShift|Shift the value using the Date-shift algorithm.|Yes|
|cryptoHash|Transform the value using Crypto-hash method.|Yes|
|encrypt|Transform the value using Encrypt method.|Yes|
|substitute|Substitute the value to a predefined value.|Yes|
|refreshUID|replace with a non-zero length UID|No|

Setting configuration indicates the algorithm needs _defaultSettings_ and _customizedSettings_ or not.

### Redact 

The value will be erased by default. But for age (AS), date (DA) and date time (DT), users can enable partial redact in setting as follow:

|Parameters|Description|Valid Value|Affected VR|Required|default value|
|----|------|--|--|--|--|
|enablePartialAgesForRedact|If the value is set to true, only age values over 89 will be redacted.|boolean| AS |False|False|
|enablePartialDatesForRedact|If the value is set to true, date, dateTime will keep year. e.g. 20210130 -> 20210101|boolean|DA, DT|False|False|

Here is a sample rule using redact method. It uses _defaultSettings_ which enables partial redact both for age, date and dateTime:
```
{
    "rules": [
        {"tag": "(0010,0020)","method": "redact"},
    ],
    "defaultSettings":[
        {"redact":{"enablePartialAgesForRedact": true","enablePartialDatesForRedact": true}}
    ],
    "cutomizedSettings":[
    ]
}

```

### Perturb

With perturb rule, you can replace specific values by adding noise. Perturb function can be used for numeric values (ushort, short, uint, int, ulong, long, decimal, double, float). Setting for perturb includes following parameters:

|Parameters|Description|Valid Value|Required|default value|
|----|----|----|----|---|
|Span| A non-negative value representing the random noise range. For fixed range type, the noise will be sampled from a uniform distribution over [-span/2, span/2]. For proportional range type, the noise will be sampled from a uniform distribution over [-span/2 * value, span/2 * value]|Positive Integer|True|1|
|RangeType|Define whether the span value is fixed or proportional. If type is fixed, the range will be [-span/2, span/2], and for proportional range, it will be [-span/2 * value, span/2 * value]. |fixed , proportional|False|proportional|
|RoundTo| specifies the number of decimal places to round to.|A value from 0 to 28|False|2|

Here is a sample rule using perturb method and using _perturbCustomerSetting_ as setting with a fixed range [-5, 5] with decimal place round to 0:
```
{
    "rules": [
        {"tag": "(0020,1010)", "method": "perturb", "settings":"perturbCustomerSetting"}
    ],
    "defaultSettings":[
        {"perturb":{ "span": "1", "roundTo": 2, "rangeType": "Proportional", "NoiseFunction" : "Uniform"}},
    ],
    "cutomizedSettings":[
        {"perturbCustomerSetting":{ "span": "10", "roundTo": 0, "rangeType": "Fixed", "NoiseFunction" : "Uniform"}},
    ]
}
```

### DateShift

With this method, the input date/dateTime/ value will be shifted within a specific range. Dateshift function can only be used for date (DA) and date time (DT) types. In configuration, customers can define dateShiftRange, DateShiftKey and dateShiftScope. 

|Parameters|Description|Valid Value|Required|default value|
|----|----|--|--|--|
|dateShiftRange| A non-negative value representing the dateshift range.|positive integer|False|100|
|dateShiftKey|Key used to generate shift days.|string|False|A randomly generated string will be used as default key|
|dateShiftScope|Scopes that share the same date shift key prefix and will be shift with the same days. |SeriesInstance, StudyInstance, SOPInstance. |False|SeriesInstance|

Here is a sample rule using dateShift method on DICOM tags with VR in DA. The dateShift setting is given in _defaultSettings_ field:
```
{
    "rules": [
        {"tag": "DA",  "method": "dateshift"}
    ],
    "defaultSettings":[
        {"dateShift":{"dateShiftKey": "123", "dateShiftScope": "SeriesInstance", "dateShiftRange": "50"}}
    ],
    "cutomizedSettings":[
    ]
}
```

### CryptoHash
This function hash the value and outputs a Hex encoded representation (for example, a3c024f01cccb3b63457d848b0d2f89c1f744a3d). The length of output string depends on the hash function (e.g. sha256 will output 64 bytes length), you should pay attention to the length limitation of output DICOM file.
In cryptoHash setting, you can set cryptoHash key and cryptoHash function (only support sha256 for now) for cryptoHash.
|Parameters|Description|Valid Values|Required|default value|
|----|------|--|--|--|
|cryptoHashKey| Key for cryptoHash|string|False|A randomly generated string|
|cryptoHashFunction| CryptoHash function |sha256|False|sha256|

Here is a sample rule using cryptoHash on DICOM tag named PatientID with default cryptoHash setting:

```
{
    "rules": [
        {"tag": "PatientID",  "method": "cryptohash"}
    ],
    "defaultSettings":[
        {"cryptoHash":{"cryptoHashKey": "123", "cryptoHashFunction": "sha256" }}
    ],
    "cutomizedSettings":[
    ]
}
```

### Encryption
We use AES-CBC algorithm to transform the value with an encryption key, and then replace the original value with a Base64 encoded representation of the encrypted value. The algorithm generates a random and unique initialization vector (IV) for each encryption, therefore the encrypted results are different for the same input values.

Users can set encrypt key in encrypt setting.
|Parameters|Description|Valid Values|Required|default value|
|----|------|--|--|--|
|encryptKey| Key for encryption|128, 192 or 256 bit string|False|A randomly generated 256-bit string|

>Note: Similar with cryptoHash function, you should use the method on those fields that accept a Base64 encoded value and avoid encrypting data fields with length limits because the Base64 encoded value will be longer than the original value.

Here is a sample rule using encrypt method on PN tags with customized setting:
```
{
    "rules": [
        {"tag": "PN", "method": "encrypt", "setting":"customizedEncryptSetting"}
    ],
    "defaultSettings":[
        "encrypt": {"encryptKey": "123456781234567812345678"},
    ],
    "cutomizedSettings":[
        "customizedEncryptSetting": {"encryptKey": "0000000000000000"},
    ]
}
```

### Substitute
Using substitue, you can specify a fixed and valid value to replace a target field. You can specify the parameter "replaceWith" in setting, which is the new value for substitute.

|Parameters|Description|Valid Values|Required|default value|
|----|------|--|--|--|
|replaceWith| New value for substitute|string|True|"ANONYMOUS"|

Here is a sample rule using substitute method on dateTime tags and replace the value to "20000101":
```
{
    "rules": [
        {"tag": "DT", "method": "substitute", "setting":"customizedDateTimeSubstituteSetting"}
    ],
    "defaultSettings":[
        "substitute": {"replaceWith": "ANONYMOUS"}
    ],
    "cutomizedSettings":[
        {"customizedDateTimeSubstituteSetting":{"replaceWith": "20000101"}},
    ]
}
```

# Usage Notes
## AutoValidation
De-Identification may transform a value to an invalid output. You can enable  AutoValidation, it validates the output for each rule. The content of DICOM items should be validated as soon as they are added to the DICOM dataset. If autoValidation is disabled, the tool will not check the length and format of string value. (Only format validation for string is disabled. For others, e.g. integer, decimal, bytes, etc, still need valid values and could not be set to other types.)

For example, if using encryption method on PatientID, which is a 64 chars maximum string, the encrypted output may exceed 64 chars. If disable autoValidation, the invalid value will be returned and the output DICOM file may be invalid for the continuing process. If you enable autoValidation, this encryption process will fail.
## Current Limitation
1. We only support metadata de-identification in DICOM. Not applicable on pixel data. 
2. For DICOM tag which is a Sequence of Items (SQ), we only support redact and remove methods on the entire sequence.

# Contributing
This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [the CLA site](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
