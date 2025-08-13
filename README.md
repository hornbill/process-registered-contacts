# Hornbill Contact Registration Processor [GO](https://golang.org/) - Script to process registered contacts in Hornbill

## Quick links

- [Overview](#overview)
- [Installation](#Installation)
- [Requirements](#Requirements)
- [Authentication](#Authentication)
- [Configuration](#Configuration)
  - [Domain Black List](#domain-black-list)
  - [Domain White List](#domain-white-list)
  - [Multi Factor Authentication](#multi-factor-authentication)
  - [Summary Auto Task](#summary-auto-task)
- [Execute](#execute)
- [Testing](#testing)
- [Logging](#logging)
- [Error Codes](#error codes)

## Overview

This tool provides functionality to automate the approval or rejection of contacts which have registered via the Customer-portal registration page (provided you haven't enabled auto-approval).

The following tasks are carried out when the tool is executed:

- The list of registered accounts is obtained from Hornbill.
- The domain name of each email address will be compared against the **blacklist** and the **whitelist**.
- IF the domain of the email address is in the blacklist, then the account will be deleted
- IF the domain of the email address is in the whitelist, then the account will be approved

## Download
The utility can be downloaded from [GitHub](https://github.com/hornbill/process-registered-contacts/releases/latest). Please ensure to download the latest version of the ZIP archive that is relevant to the architecture of the computer it will be executed and scheduled to run on.

## Installation

### Windows

- Download the archive containing the import executables
- Extract zip into a folder you would like the application to run from e.g. `C:\hornbill_contact_processing\`
- Open '''conf.json''' and add in the necessary configuration
- Open Command Line Prompt as Administrator
- Change Directory to the folder containing the extracted files `C:\hornbill_contact_processing\`
- Run the command relevant to the computer you are running this on:
  - process-registered-contacts.exe -dryrun=true


## Requirements

### Host

The program that performs the import is fairly lightweight and doesn’t require much in the way of hardware to run. It can be run on virtualized or physical hardware running any version of Windows currently supported by Microsoft, but basic guidelines are as follows:

- Operating System - Microsoft Windows, 32 or 64-bit, current/LTS, desktop/server
- CPU - Intel-compatible, one or more cores
- RAM - 4GB minimum

### Network

The utility connects to the Hornbill instance in the cloud over HTTPS/SSL, so as long as you have standard internet access then you should be able to use it without the need to make any proxy or firewall configuration changes.

####HTTP Proxies

If you use a proxy for your internet traffic, the HTTP_PROXY and HTTPS_PROXY environment variables will need to be set. These environment variables hold the hostname or IP address of your proxy server. The proxy environment variables can be set from a command line as so:
```
cmd
set HTTP_PROXY=HOST:PORT
set HTTPS_PROXY=HOST:PORT
```
Where “HOST” is the IP address or hostname of your Proxy Server and “PORT” is the specific port number. If you require a username and password to go through the proxy, the format for the setting is as follows:
```
cmd
set HTTP_PROXY=username:password@HOST:PORT
set HTTPS_PROXY=username:password@HOST:PORT
```

#### URL White Listing

Occasionally, on top of setting the HTTP_PROXY and HTTPS_PROXY variables, the following URLs may need to be white-listed to allow access out from your network to the required endpoints in the Hornbill network:

- https://*.hornbill.com/* - allows access to the required Hornbill instance information and API endpoints
- https://api.github.com/repos/hornbill/* - allows the utility to self-update

## Authentication


### API Key Rules
```
admin:contactDelete
admin:portalAccountGetContacts
admin:portalSetContactAccess
bpm:autoTaskRun
data:entityUpdateRecord
system:logMessage
```


## Configuration

Example JSON File:

```json
{
	"DomainBlackList": [ "domain1.com" ]
	, "DomainWhiteList": [ "domain2.com" ]
	, "MFA": 0
	, "SummaryAutoTask": ""
}
```

### Domain Black List

`DomainBlackList` is an array of strings, listing any **blacklisted** domains.
Registered contacts with email addresses from these domains will be **deleted**.

#### Domain White List

`DomainWhiteList` is an array of strings, listing any **whitelisted** domains.
Registered contacts with email addresses from these domains will automatically be **approved** with their email address set as their login id and taking on the setting of MFA as determined in the configuration file.

#### Multi Factor Authentication

`MFA` - What form of Authentication the registered contact should use.

- 0: Disabled *(default)* - the password set on registration is to be used.
- 1: Email
- 2: Authentication App


#### Summary Auto Task

`Summary Auto Task` - **string** - The name of the ( *servicemanager* ) **Global** AutoTask to be triggered.

The AutoTask will need the following Inputs:

- `dryrun` - **boolean** - whether the utility is run in dryrun mode. One would probably want to put some logic in the AutoTask to ignore dryruns.
- `errorCount` - **integer** - amount of errors which occurred during the running of the utility.
- `skipCount` - **integer** - amount of registered contacts which were skipped (i.e. are still registered - and to be manually progressed). One would probably want to monitor this, for instance notifying an individual if there are (a certain amount of) results here.
- `removalCount` - **integer** - amount of registered contacts which have been removed (because their domains were blacklisted).
- `registeredCount` - **integer** - amount of registered contacts which have been approved (because their domains were whitelisted).


## Execute

### Command Line Parameters

- file - Defaults to `conf.json` - Name of the Configuration file to load
- dryrun - Defaults to `false` - Set to True and there won't be any changes to the contacts. The log file WILL contain what the utility would have done, this is to aid in debugging the initial connection information.
- debug - Defailts to `false` - set to true to increase debug logging output
- concurrent - defaults to `1`. This is to specify the number of requests that should be imported concurrently, and can be an integer between 1 and 10 (inclusive). 1 is the slowest level of import, but does not affect performance of your Hornbill instance, and 10 will process the import much more quickly but could affect performance.
- version - shows version of tool (and ends)
- logprefix - Add prefix to log file
- creds - defaults to `false` - Returns stored API Key and ends")

### Testing

If you run the application with the argument dryrun=true then no requests will be logged - the XML used to raise requests will instead be saved in to the log file so you can ensure the data mappings are correct before running the import.

'process-registered-contacts.exe -dryrun=true'

### Logging

All Logging output is saved in the log directory in the same directory as the executable the file name contains the date and time the import was run 'SW_Call_Import_2015-11-06T14-26-13Z.log'

### Error Codes

- `100` - Unable to create log File
- `101` - Unable to create log folder
- `102` - Unable to Load Configuration File
