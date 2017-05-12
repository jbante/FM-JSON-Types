# FileMaker JSON Type Handling

FileMaker 16 introduced [a collection of built-in functions][1] for manipulating data serialized as JSON. This makes it easier for FileMaker applications to interact with many web services. This will also make JSON the de facto standard format for scripts within FileMaker to pass parameters and results to each other, improving code sharing within the FileMaker community.

[1]: https://fmhelp.filemaker.com/help/16/fmp/en/#page/FMP_Help/json-functions.html "FileMaker documentation on JSON functions"

JSON does not have a broad palette of scalar data types to choose from: text, number, boolean, and null. On top of that, FileMaker's JSONGetElement function always returns a text result, even when the serialized JSON value is a number or boolean. Sending and receiving data with correct types requires more logic on top of the new JSON functions.

This module uses [ISO 8601 format][2] for dates, times, and timestamps. This is the most popular method for representing such data used by web services, so this module is also useful for integrating web services with FileMaker applications.

[2]: https://en.wikipedia.org/wiki/ISO_8601 "ISO 8601 on Wikipedia"

This module supports two approaches for translating between FileMaker data types and JSON data:
1. Custom functions - This approach is convenient to read and write. However, copying logic using the custom functions from one file to another is more complicated, since the custom functions must also be copied, and they must be copied _before_ any dependent logic.
2. Scripts - This approach is convenient to copy and paste between FileMaker files as part of a module script folder. However, code using these scripts for data translation is more cumbersome than using the custom functions.

## Installation

1. Import all the custom functions from the FileMaker file  into your application. You can skip this step if you only want to use scripts to translate typed data between FileMaker and JSON.
2. Import the "Modules" / "FM-JSON Types" script folder and all its contents from the FileMaker file into your application.

## How to use the custom functions

###### Set a number:

	JSONSetElement ( "{}" ; "numberKey" ; 1.23e+4 ; JSONNumber )
	= {"numberKey":1.23e+4}

###### Get a number:

	JSONGetNumber ( $json ; "numberKey" )
	= 1.23e+4

Since JSON supports number-type data, no function is necessary when serializing to JSON.

###### Set a boolean:

	JSONSetElement ( "{}" ; "booleanKey" ; False ; JSONBoolean )
	= {"booleanKey":false}
	
###### Get a boolean:

	JSONGetBoolean ( $json ; "booleanKey" )
	= 0

Since JSON supports boolean-type data, no function is necessary when serializing to JSON.

###### Set a date:

	JSONSetElement ( "{}" ; "dateKey" ; ISOFromDate ( Date ( 5 ; 9 ; 2017 ) ) ; JSONString )
	= {"dateKey":"2017-05-09"}

###### Get a date:

	JSONGetDateFromISO ( $json ; "dateKey" )
	= 9 May 2017

Dates are formatted according to ISO 8601 and serialized as JSONStrings.

###### Set a time:

	JSONSetElement ( "{}" ; "timeKey" ; ISOFromTime ( Time ( 18 ; 34 ; 56.7 ) ) ; JSONString )
	= {"timeKey":"18:34:56,7"}

###### Get a time:

	JSONGetTimeFromISO ( $json ; "timeKey" )
	= 6:34:56.7 pm

Times are formatted according to ISO 8601 and serialized as JSONStrings. ISO 8601 supports time zones, but FileMaker times do not. These functions will ignore time zone information from other sources.

###### Set a timestamp:

	JSONSetElement ( "{}" ; "timestampKey" ; ISOFromTimestamp ( $timestamp ) ; JSONString )
	= {"timestampKey":"2017-05-09T12:34:56,7"}

###### Get a timestamp:

	JSONGetTimestampFromISO ( $json ; "timestampKey" )
	= 9 May 2017, 6:34:56.7 pm

Timestamps are formatted according to ISO 8601 and serialized as JSONStrings. ISO 8601 supports time zones, but FileMaker timestamps do not. These functions will ignore time zone information from other sources.

###### Set a container:

	JSONSetElement ( "{}" ; "containerKey" ; JSONContainerObject ( $container ) ; JSONObject )
	= {"containerKey":{"base64":"iVBOR...","fileName":"image.png"}}

###### Get a container:

	JSONGetContainer ( $json ; "containerKey" )

Containers are serialized as JSONObjects with sub-values for the file name and the base 64-encoded binary data.

## How to use the scripts

###### Set a number:

	Perform Script [ "Your Script" ; Parameter: JSONSetElement ( "{}" ; "numberKey" ; 1.23e+4 ; JSONNumber ) ]

###### Get a number:

	Set Variable [ $number ; Value: GetAsNumber ( JSONGetElement ( Get ( ScriptResult ) ; "numberKey" ) ) ]

Since FileMaker has a built-in `GetAsNumber` function, there is no value in using a separate script to interpret number data from JSON.

###### Set a boolean:

	Perform Script [ "Your Script" ; Parameter: JSONSetElement ( "{}" ; "booleanKey" ; False ; JSONBoolean ) ]

###### Get a boolean:

	Set Variable [ $boolean ; Value: GetAsBoolean ( JSONGetElement ( Get ( ScriptResult ) ; "booleanKey" ) ) ]

Since FileMaker has a built-in `GetAsBoolean` function, there is no value in using a separate script to interpret boolean data from JSON.

###### Set a date:

	Perform Script [ "Convert to ISO 8601 from Date" ; Parameter: Date ( 5 ; 9 ; 2017 ) ]
	Set Variable [ $json ; Value: JSONSetElement ( "{}" ; "dateKey" ; Get ( ScriptResult ) ; JSONString ) ]

###### Get a date:

	Perform Script [ "Convert to Date from ISO 8601" ; Parameter: JSONGetElement ( $json ; "dateKey" ) ]
	Set Variable [ $date ; Value: Get ( ScriptResult ) ]

Dates are formatted according to ISO 8601 and serialized as JSONStrings.

###### Set a time:

	Perform Script [ "Convert to ISO 8601 from Time" ; Parameter: Time ( 18 ; 34 ; 56.7 ) ]
	Set Variable [ $json ; Value: JSONSetElement ( "{}" ; "timeKey" ; Get ( ScriptResult ) ; JSONString ) ]

###### Get a time:

	Perform Script [ "Convert to Date from ISO 8601" ; Parameter: JSONGetElement ( $json ; "timeKey" ) ]
	Set Variable [ $time ; Value: Get ( ScriptResult ) ]

Times are formatted according to ISO 8601 and serialized as JSONStrings. ISO 8601 supports time zones, but FileMaker times do not. These scripts will ignore time zone information from other sources.

###### Set a timestamp:
 
	Perform Script [ "Convert to ISO 8601 from Time" ; Parameter: $timestamp ]
	Set Variable [ $json ; Value: JSONSetElement ( "{}" ; "timestampKey" ; Get ( ScriptResult ) ; JSONString )

###### Get a timestamp:

	Perform Script [ "Convert to Date from ISO 8601" ; Parameter: JSONGetElement ( $json ; "timestampKey" ) ]
	Set Variable [ $timestamp ; Value: Get ( ScriptResult ) ]

Timestamps are formatted according to ISO 8601 and serialized as JSONStrings. ISO 8601 supports time zones, but FileMaker timestamps do not. These scripts will ignore time zone information from other sources.

###### Set a container:

	Perform Script [ "Convert to JSONObject from Container" ; Parameter: $container ]
	Set Variable [ $json ; Value: JSONSetElement ( "{}" ; "containerKey" ; Get ( ScriptResult ) ; JSONObject ) ]

###### Get a container:

	Perform Script [ "Convert to Container from JSONObject" ; Parameter: JSONGetElement ( $json ; "containerKey" ) ]
	Set Variable [ $container ; Value: Get ( ScriptResult ) ]

Containers are serialized as JSONObjects with sub-values for the file name and the base 64-encoded binary data.

## License

Anyone may do anything with this software for any purpose. There is no warranty.