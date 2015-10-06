# SUPPORT TO NEW SW VERSION IN BUILD CLASS 

- Description
  - An extra field is needed to be provided by class Build exposing the current SW version in a specific format.
- Requirements


##### OEM must:
  - Create a new class com.xxx.os.Build extending android.os.Build
  - The class must be provided in library com.xxx.os
  - This new class must add field public static final String SW_VERSION
  - This field must be populated with a string identifying the current SW version
  - The string format must be: `<Android release version>/<Android build string>/<OEM string>`


##### Rules for `<OEM string>`
  - It cannot contain slash ("/") character
  - It must be formatted as follows: `<Vendor name><sp><some vendor specific string>`


##### *Example: 4.4.2/KOT49H/OEM ABCDEFG1234*



## Description and implementation


To comply with the requirements, we create a new class com.xxx.os.Build as follows.

- frameworks/base/core/java/com/xxx/os/Build.java

```javascript
package com.xxx.os;
import android.os.SystemProperties;

public class Build extends android.os.Build
{
    /**create a new field to identify the current software version */
    public static final String SW_VERSION = deriveSW_VERSION();

    /**
    * The string format must be: <Android release version>/<Android build string>/<OEM string>.
    */
    public static String deriveSW_VERSION() {
        String version = getString("ro.build.version.release") + '/' +
                         getString("ro.build.id") + '/' +
                         getString("ro.product.oem") + ' ' +
                         getString("ro.build.string");
        return version;
    }   

    private static String getString(String property) {
        return SystemProperties.get(property, UNKNOWN);
    }   
}
```

From above, we summarize:

1. Create a new class com.xxx.os.Build that extends android.os.Build
 - `public class Build extends android.os.Build`

...On an AOSP source code, Build.java is located at path framework/base/core/java/android/os/Build.java. In order to create a new class Build that extends existed one, we create a new class at path framework/base/core/java/com/xxx/os/Build.java and extends the old one.


2. The class must be provided in library com.xxx.os
 - `package com.xxx.os;`

...The class has to be provided at library com.xxx.os, so we use package to create the class under the package.

3. Add field public static final String SW_VERSION whose format must be formatted as `<Android release version>/<Android build string>/<OEM string>`. Also, `<OEM string>` has to be formatted as `<Vendor name><sp><some vendor specific string>`.
   
   
 - `public static final String SW_VERSION = deriveSW_VERSION();`

...We declare a public static final String SW_VERSION and call the method deriveSW_VERSION() to comply with the required format.
   
 - ` public static String deriveSW_VERSION() {
           String version = getString("ro.build.version.release") + '/' +
	                    getString("ro.build.id") + '/' +
			    getString("ro.product.oem") + ' ' +
                            getString("ro.build.string");
           return version;
	}`

...From the method deriveSW_VERSION(), we can derive the string from the system properties defined at build/tool/buildinfo.sh (https://android.googlesource.com/platform/build/+/master/tools/buildinfo.sh)..
    
...NOTE: we define ro.build.string to set vendor specific string. We describe it below...



##### To get vendor specific string, we declare **ro.build.string** and assign a variable **OEM_VENDOR_STRING** at path build/tools/buildinfo.sh. Also, we have to define the variable as follows.

- build/tools/buildinfo.sh
``` sh
echo "ro.build.string=$OEM_VENDOR_STRING"
```

- build/core/Makefile
``` sh
OEM_VENDOR_STRING=ABCDEFG1234
```

``` sh
   $(hide) TARGET_BUILD_TYPE="$(TARGET_BUILD_VARIANT)" \
            OEM_TYPE="$(OEM_TYPE)" \
            TARGET_DEVICE="$(TARGET_DEVICE)" \
            PRODUCT_NAME="$(TARGET_PRODUCT)" \
            PRODUCT_BRAND="$(PRODUCT_BRAND)" \
            PRODUCT_DEFAULT_LANGUAGE="$(call default-locale-language,$(PRODUCT_LOCALES))" \
            PRODUCT_DEFAULT_REGION="$(call default-locale-region,$(PRODUCT_LOCALES))" \
            PRODUCT_DEFAULT_WIFI_CHANNELS="$(PRODUCT_DEFAULT_WIFI_CHANNELS)" \
            OEM_VENDOR_STRING="$(OEM_VENDOR_STRING)" \
            PRODUCT_MANUFACTURER="$(PRODUCT_MANUFACTURER)" \
            PRIVATE_BUILD_DESC="$(PRIVATE_BUILD_DESC)" \
            BUILD_ID="$(BUILD_ID)" \
            BUILD_DISPLAY_ID="$(BUILD_DISPLAY_ID)" \
            BUILD_NUMBER="$(BUILD_NUMBER)" \
            ...
            bash $(BUILDINFO_SH) >> $@
```

Then, we can create a new class and define the required fields or methods.


##### *** How to vadicate our implement does work? Here, we do not write an extra application to test the result. Instead, we use the existed code about settings to import our new class and replace some output string to show if it work. Just for your reference.


- packages/apps/Settings/src/com/android/settings/deviceinfo/SoftwareInformationFragment.java

``` javascript
...
import android.content.pm.ResolveInfo;
import android.os.Binder;
//import android.os.Build;
import com.verizon.os.Build;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
...
public void onCreate(Bundle icicle) {
    super.onCreate(icicle);
    addPreferencesFromResource(R.xml.device_info_software);
    PackageManager pm = getPackageManager();
    setValueSummary(KEY_BASEBAND_VERSION, "gsm.version.baseband");
    String DeviceInfoDefault = getResources().getString(R.string.device_info_default);
    //for CTA: Kernel version: Append UD when userdebug
    //         Build number: BSP customized
    if (isCTA() && Utils.isCNSKU()) {
        setStringSummary(KEY_KERNEL_VERSION, getFormattedKernelVersion().concat(Build.TYPE.equals("userdebug") ? " " + Build.SW_VERSION : ""));
        setStringSummary(KEY_BUILD_NUMBER, ResCustomizeConfig.getProperty(KEY_BUILD_NUMBER, DeviceInfoDefault));
    } else {
        setStringSummary(KEY_KERNEL_VERSION, getFormattedKernelVersion());
        setStringSummary(KEY_BUILD_NUMBER, generateFormatBuildNumber());
    }   
    findPreference(KEY_BUILD_NUMBER).setEnabled(true);

    if (!SELinux.isSELinuxEnabled()) {
        String status = getResources().getString(R.string.selinux_status_disabled);
        setStringSummary(KEY_SELINUX_STATUS, status);
    } else if (!SELinux.isSELinuxEnforced()) {
        String status = getResources().getString(R.string.selinux_status_permissive);
        setStringSummary(KEY_SELINUX_STATUS, status);
    }   

    // Remove selinux information if property is not present
    setPropertySummary(getPreferenceScreen(), KEY_SELINUX_STATUS,
        PROPERTY_SELINUX_STATUS);

    // Remove Baseband version if wifi-only device
    if (Utils.isWifiOnly(getActivity())) {
        getPreferenceScreen().removePreference(findPreference(KEY_BASEBAND_VERSION));
    }
}
```

The settings print the required string format successfully.

![Image of Settings] (https://github.com/GerardHaugan/Android/blob/master/build_system/Settings.png)

