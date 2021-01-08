# De-id functions for CTP
## Element Functions
|Functions|CTP|Fhir|Description|
|-----|------|-----|------|
|keep|keep()|keep|Keep both for element and sequence.|
|redact|remove()|redact|Redect element.|
|substitute|contents(ElementName)<br> contents(ElementName,"regex")<br>contents(ElementName,"regex","replacement")<br>lookup(ElementName,KeyType)<br> lookup(ElementName,KeyType,action)|substitute|Fhir can only replace the whole field with a given value. CTP could replace part of the value.|
|encrypt|encrypt(ElementName,"key")<br> encrypt(ElementName,@ParameterName)|encrypt|CTP has flexible encrypt key.|
|hash|hash(ElementName)<br> hash(ElementName,maxCharsOutput) <br>hashname(ElementName,maxCharsOutput) <br>hashname(ElementName,maxCharsOutput,maxWordsInput)<br>hashptid(siteID,ElementName)<br> hashptid(siteID,ElementName,maxCharsOutput) <br> hashuid(root,ElementName) <br> hashuid(root,ElementName,ElementName2)|cryptohash|Hash methods are different for each field in CTP.|
|dateshift|hashdate(ElementName,HashElementName) <br> incrementdate(ElementName,incInDays)|dateshift|Offset for datashift could be fixed or random in CTP.|
|generalize|round(ElementName,groupsize)<br>truncate(ElementName,n) <br>modifydate(ElementName,year,month,day)|generalize|Fhir could be more flexible by using expression.|
|append|append()(script)|-|Adds the value of a script to a multi-value element.|
|blank|blank(n)<br> empty()|substitute|Returns a string of blanks of length n.<br>returns a zero-length string.|
|initials|initials(ElementName)<br>initials(ElementName, offset)||Returns first letter of the content.|
|integer|integer(ElementName,KeyType,width)||Replacement strings start at 1 and increment for each new value of the named element.|
|others|always()<br> call(id, args)<br>date(separator)<br> dateinterval()<br>lowercase(ElementName) <br>param(@ParameterName) <br>pathelement(ElementName,index)<br>process()<br>require()<br>time(separator)<br>uppercase(ElementName)<br>value(ElementName)||Could be combined with other functions.|


## Global Functions

* Keep group 18
* Keep group 20
* Keep group 28
* Keep safe private elements
* Remove private groups
* Remove unchecked elements
* Remove curves
* Remove overlays

## Conditional Functions

* if(ElementName,exists)
* if(ElementName,isblank)
* if(ElementName,equals,"string")
* if(ElementName,contains,"string")
* if(ElementName,matches,"regex")
* if(ElementName,greaterthan,value)
* quarantine( )
* select( )
* skip( )

# Compared With Fhir
pros:
1. More de-id functions on element are provided.
2. Some Functions could be combined. e.g. @always()@date().
3. Provide global functions and conditional functions.

cons:
1. A third part tool needs integration with our server.
2. User should edit the CTP script which is different with Fhir's configuration file. This inconsistence could cause the bad experience.