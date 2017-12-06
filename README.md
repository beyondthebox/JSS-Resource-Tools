# JSS Resource Tools

JSS Resource Tools is a command line utility written in Python that utilises the Jamf Pro (previously Casper Suite) API in order to import, export and update JSS resources en-masse. Whilst its purposes are many, popular examples include:

 - Exporting mobile device or computer records to CSV (with custom fields, primary keys & extension attributes)
 - Mass import of users
 - Mass assignment of mobile device or computer records to users
 - Mass update of mobile device or computer custom fields & extension attributes
 - Automated synchronisation of JSS data with external asset management or similar systems

JSS Resource Tools is developed and maintained by [Beyond the Box](https://www.beyondthebox.com.au). We first developed this script to assist with large scale 1:1 Mac & iOS rollouts into schools where traditional directory user assignment was not being utilised. Through many iterations over a few years, it has helped us build and manage successful Casper and Jamf Pro deployments for our customers.

You can learn more about use cases and examples of JSS Resource Tools at [Beyond the Box Labs](https://beyondthebox.github.io).

## JSS Requirements

JSS Resource Tools utilises the JSS API and therefore requires an account with API read and write access. It checks for successful authentication but does not currently do any specific authorisation checks, so assumes the account you give it has appropriate privileges. It has been tested and works with:

- Casper Suite 9.8 and greater
- Jamf Pro 10

## Installing JSS Resource Tools

For requirements and install instructions, see [INSTALL](https://github.com/beyondthebox/JSS-Resource-Tools/blob/master/INSTALL.md).

## Exporting Resources

Whilst basic export functionality is built into the JSS, there can be limited flexibility in terms of the fields you can request for export, particularly around keying to other resources. By using `--mode export`, the tool provides extremely flexible export functionality. In more advanced usage, this tool can export resource foreign keying (ie. exporting the mobile device ID/s for each user) or exporting fields that are not easily exportable using the JSS interface (i.e exporting the current version of GarageBand installed on each mobile device).

When exporting a CSV of supported resources, the script will include a default set of fields. You can request export of nearly every other field by supplying 1 or more `--field` options to the script. The script uses the logic expressed in the Field Naming section below to match and export fields from a JSS resource. Common fields are also listed for your convenience in the Field Listing section at the bottom of this document.

#### Performance
A quick note on export performance: it's not going to set your world on fire. Due to API limitations and the tool's flexibility in allowing almost any custom field to be exported, each resource must be independently requested and processed as a separate and independent HTTP call. This will mean that larger resource sets take a while to export.
 
## Importing Resources

By using `--mode import`, a CSV file can be passed for validation and processing. Properly formatted records will be sent to the JSS API for resource creation.

### Users

This tool can easily and safely import user records from CSV, and can facilitate advanced user and group scoping in Jamf Pro where a tradtional directory connection (LDAP, Active Directory) does not exist. These user records can then be updated and keyed against mobile devices or computers, allowing you to assign hardware to users.

## Updating Resources

One of the tool's best and most powerful features is its ability to update existing records in the JSS from a CSV input file. This mode (`--mode update`) can be used to make quick and consistent updates to resources; matching on resource ID or almost any other unique key available.

Popular use cases include:

- Assigning computers or mobile devices to users
- Updating mobile device or computer names
- Updating asset tags or purchasing information
- Updating or assigning extension attributes against resources


### Field Naming

Field naming is how you get data in and out of the JSS using the tool. Almost any field should be able be processed on a JSS resource using custom field naming in your CSV file. The script matches fields using XML XPath conventions, with some modified helpers for resource extension attributes.

The JSS API returns resource records via XML, and stores fields within different children in a resource's tree structure. In order to simplify this concept for easy compatibility with CSV headers, JSS resource tools transparently maps fields using:

- Field names in root element
- Field via Xpath in child elements
- Extension attributes helper

#### Xpath (default)

The tool uses Xpath to match to fields within a resource XML object. This allows for incredibly precise and advanced scoping of fields within a resource:

To get the current IP address for a computer or mobile device:

`general/ip_address`

To get warranty expiry for a computer or mobile device:

`purchasing/warranty_expires`

To get the number of applications installed on a mobile device:

`applications/size`

To get the display name of the first configuration profile installed on a mobile device:

`configuration_profiles/*[1]/display_name`

To get the current version of Pages on a mobile device:

`applications/application/[identifier="com.apple.Pages"]/application_version`

#### Extension Attributes

The tool fully supports export, import and updates of extension attributes on JSS resources.

In order to make extention attributes easier to work with, you can format field names with a helper syntax. The syntax is as follows:

`extension_attribute/{id}`

For example, if you have a mobile device extention attribute called "Case Type" and with an ID of "2" that you wish to update, you would name your field:

`extension_attribute/2`

You can get a listing of valid extention attributes and their IDs for a particular resource by printing an Example Resource Record. See "Example Resource Record".


### Common Fields

Following is a list of common fields or interesting examples that are available through field naming:

#### Users

| Fieldname | Description |
|-----------|-------------|
| `extension_attribute/1` | Value of the extention attribute with ID# 1 for the user. |
| `links/mobile_devices/*[0]/id` | ID of the first mobile device assigned to the user. |
| `links/computers/*[0]/id` | ID of the first mobile device assigned to the user. |

#### Mobile Devices

| Fieldname | Description |
|-----------|-------------|
| `general/name` | The mobile device name. |
| `general/asset_tag` | The mobile device asset tag. |
| `general/ip_address` | The last reported IP address for the device. |
| `general/ip_address` | The last reported IP address for the device. |
| `location/username` | The username assigned to the device. |
| `purchasing/is_purchased` | Is device purchased? Takes TRUE or FALSE value. |
| `purchasing/is_leased` | Is device leased? Takes TRUE or FALSE value. |
| `purchasing/po_number` | Purchase Order Number |
| `purchasing/vendor` | Vendor Name |
| `purchasing/purchase_price` | Purchase Price |
| `applications/size` | The number of applications installed on the device. |

## Command Examples

Following is a list of command examples. For cleanliness, they omit the `--jss`, `--username` and `--password` options that you may generally provide in the command. In these examples, you will be prompted for them once running the command.

`jss_resource_tools.py --mode example --type mobiledevices`

Prints an example XML record of a mobile device from your JSS.

`jss_resource_tools.py --mode import --type users UsersToImport.csv`

Imports the users defined in the file "UsersToImport.csv" into the JSS. In this example, "UsersToImport.csv" would look like:

<img src="https://www.beyondthebox.com.au/assets/images/jssresourcetools/csv_example_1.png" width="430">

`jss_resource_tools.py --mode export --type computers Export.csv`

Exports the default fields for computer resource objects into the file "Export.csv".

`jss_resource_tools.py --mode export --type mobiledevices --field general/ip_address --field location/username Export.csv`

Exports the default fields, last known IP address and assigned user for mobile device resource objects into the file "Export.csv".

`jss_resource_tools.py --mode export --type mobiledevices --field general/ip_address --field location/username Export.csv`
Exports the default fields, last known IP address and assigned user for mobile device resource objects into the file "Export.csv".

`jss_resource_tools.py --mode update --type computers MobileDevices.csv`
Updates existing enrolled computers by keying on their resource "id", and updating other fields defined in the CSV file "MobileDevices.csv". In this example; in order to update the Vendor and PO Number for devices, "MobileDevices.csv" would look like:

<img src="https://www.beyondthebox.com.au/assets/images/jssresourcetools/csv_example_3.png" width="320">

`jss_resource_tools.py --mode update --type mobiledevices --match serial_number MobileDevices.csv`

Updates existing enrolled mobile devices by matching on their serial number, and updating other fields defined in the CSV file "MobileDevices.csv". In this example; in order to update device name and assign the device to a username, "MobileDevices.csv" would look like:

<img src="https://www.beyondthebox.com.au/assets/images/jssresourcetools/csv_example_2.png" width="320">

## Dry Run & Verbosity
In order to better understand the tool's behavior, as well as protect your JSS data from unintentional edits, the tool supports both `--verbose` and 	`--dry` options.

`--verbose` significantly increases the amount of output and logging that the tool produces, and will help of you are experiencing issues or errors. 

`--dry` toggles a "Dry Run" of the tool. This runs through standard processing of fields and elements but skips any actual writes or updates to your JSS. This is very useful in ensuring that your CSV formatting and field naming is correct before making production imports or updates to your JSS.

## Example Resource Record

In order to more easily determine field names and structure for a resource for import or updating, you can use `--mode example` to fetch an example resource. This will simply pretty print the XML structure of a random resource returned by the API. See the "Field Naming" section above for details on how to use this information to update resource records.

## Danger Zone

Can you break enrollments using this script? Absolutely.

When updating mobile devices and computers using the script, changing the hardware UDID in the JSS can break enrollment. For this reason, the script will refuse to update any field name matching `"udid"`. If you really know what you are doing, you can override this safety feature by supplying the `--danger` flag. 

User beware.

## License & Support

JSS Resource Tools is licensed under The MIT License, available here - [LICENSE.md](https://github.com/beyondthebox/JSS-Resource-Tools/blob/master/LICENSE.md). As per the license, this software is provided "as-is" and is not actively supported or warranted by Beyond the Box.

## About Beyond the Box

Beyond the Box are a Jamf specialist and IT support provider based in Melbourne, Australia. We specialise in seamless deployments of Apple technology into schools and businesses large and small. To learn more, [visit us here](https://www.beyondthebox.com.au).
