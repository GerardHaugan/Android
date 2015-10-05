SUPPORT TO NEW SW VERSION IN BUILD CLASS

Description

An extra field is needed to be provided by class Build exposing the
current SW version in a specific format.

Requirements

OEMs must:

• Create a new class com.verizon.os.Build extending
android.os.Build

o The class must be provided in library com.verizon.os

• This new class must add field public static final String
SW\_VERSION

• This field must be populated with a string identifying the
current SW version

• The string format must be: <Android release version\>/<Android
build string\>/<OEM string\>.

• Rules for <OEMString\>

o It cannot contain slash ("/") character

o It must be formatted as follows: <Vendor name\><sp\><some vendor
specific string\>



Example: 4.4.2/KOT49H/OEM ABCDEFG1234







Applicable to: [Smart Phone, Tablet] - Scope: [Branded]



![image](SUPPORT_html_m6b601c60.png)



