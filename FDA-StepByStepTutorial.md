# Step By Step Tutorial For Design Automation

### Step1 OAuth:
This request to the Authentication API will obtain a token with the scope require for this tutorial.
This token will be used by all the subsequent request inside the "Authorization" header to identify your forge app. This token will expire after one hour so you might need to execute the request again to get a new token if your expire.

Before executing this request, you will need to enter the forge app client id and client secret that was obtained by following the "Creating ForgeApp" section of this readme.md inside the body of the request. Once the request is completed, postman will save the access_token received in the body response inside a variable named "accessToken" to be reused in the subsequent requests.

### Step 2 Publish Bundle:

An appbundle is the package of binaries and supporting files which make your AutoCAD Addin application.
A typical zip file is zip package of your addin.bundle

#### Bundle Structure
```
\---UpdateDWGParam.bundle
    |   PackageContents.xml
    |
    \---Contents
            Newtonsoft.Json.dll
            UpdateDWGParam.dll
```
#### PackageContents.xml
```xml
<?xml version="1.0" encoding="utf-8" ?>
<ApplicationPackage SchemaVersion="1.0" Version="1.0" ProductCode="{F11EA57A-1E7E-4B6D-8E81-986B071E3E07}" Name="AutoCADDesignAutomation" Description="Sample Plugin for AutoCAD" Author="learnforge.autodesk.io>">
  <CompanyDetails Name="Autodesk, Inc" Url="http://learnforge.autodesk.io" Email="forge.help@autodesk.com"/>
  <Components>
    <RuntimeRequirements OS="Win64" Platform="AutoCAD" SeriesMin="R23.0" SeriesMax="R23.0"/>
    <ComponentEntry AppName="UpdateWindowParameters" ModuleName="./Contents/UpdateDWGParam.dll" AppDescription="AutoCAD .NET App to update parameters of Dynamic blockreference in AutoCAD Drawing" LoadOnCommandInvocation="True" LoadOnAutoCADStartup="True">
      <Commands GroupName="FPDCommands">
        <Command Global="UpdateParam" Local="UpdateParam"/>
      </Commands>
    </ComponentEntry>
  </Components>
</ApplicationPackage>
```
### Step 3: CreateBundleAlias
Creates a new alias for this AppBundle. In order to be able to use a version of an AppBundle, you have to specify an Alias for it. You can view an Alias as a user-friendly name for a specific AppBundle version.

### Step 4: Create Activity

This request to the Design Automation V3 API will create an activity. In design automation an activity specify an action that can be executed using a specified engine.


In the Design Automation API an activity specify an action that can be executed using a specified engine. The request body contains the following fields.

`id`
The id of the activity to be created. This should be unique for a given forge app.

`commandLine`
This define the command line to be executed. You use variable that will be replaced before executing your command line. This mechanism let you replace file path(s) value in command line that you tested locally with file path(s) that will make sense once the execution is launched in the Design Automation environment.

Here are some example of variables can be used:

``$(engine.path)`` will be replaced by the path where the engine is installed.

`$(args[myParam1].path)` for a given parameter with the key "myParam1", this will be replaced by the file path where the file got uploaded/downloaded to. If the parameter is a zip, it will be replaced by the path to the folder where the zip got unzipped.

`$(args[myStringParam].value)` will be replaced by the string value provided at work item submission of the parameter with the key "myStringParam" (Only for argument using the "read" verb).

`$(appbundles[myAppBundle].path)`will be replaced by the path where the "myAppBundle" app bundle have been unzipped.

`$(setting[settingX].path)` will write to a file the value of the setting with "settingX" key and be replaced by the path to the file that was written.

`$(setting[settingX].value)` will be replaced by the value of the setting with the "settingX" key.

#### engine
This define the processing engine to be installed on the machine that will execute the action. (Auto CAD, Inventor, Revit, 3ds Max)

#### parameters
These define the inputs and outputs that will need to be provided to execute the action defined.

#### inputFile
For this tutorial we define an input parameter that will be used to downloaded

#### inputJson
For this tutorial we define an input parameter json that will copied in to params.json at Forge backend.
Example:

This input parameter provided in Workitem, values `{\"Width\":\"40\",\"Height\":\"80\"}"` will be copied in localfile name mentioned in this activity i.e.s param.json.

```
"inputJson": {
			"url": "data:application/json,{\"Width\":\"40\",\"Height\":\"80\"}"
		},
```
#### outputFile
For this tutorial we define an output parameter that will put the file to given url

### Step 5: Create Activity Alias
When submitting an activity to be executed to the Design Automation API V3, we must reference it by it's alias. An alias is like a tag that point to a particular version of an activity. The version an alias point to can be changed in the future as develop more version of a given activity. The request number "2). Create Activity" created the version 1 of our activity "UpdateParamters". This request will now create an alias named "dev" to reference version 1.

### Step 6: UploadToForge

The `$.formData` in response retrieved in previous request number `4) Publish Bundle` is passed in this current request.

Uploadload url for addin package - `https://dasprod-store.s3.amazonaws.com`


For the json value `$.formData` 

```
{{ formData  }}

```


And a, typical request in curl is 
```bash
curl --request POST \
  --url https://dasprod-store.s3.amazonaws.com/ \
  --header 'content-type: multipart/form-data; boundary=---011000010111000001101001' \
  --form key=apps/uLIUN5bLOzQCmoEARUNfG0g1Nr66hj7S/ForgeTestBundleToday/1 \
  --form content-type=application/octet-stream \
  --form policy=eyJleHBpcmF0aW9uIjoiMjAxOS0wNy0wNFQwNzoxNjo1Mi43Nzc0MjI1WiIsImNvbmRpdGlvbnMiOlt7ImtleSI6ImFwcHMvdUxJVU41YkxPelFDbW9FQVJVTmZHMGcxTnI2NmhqN1MvRm9yZ2VUZXN0QnVuZGxlVG9kYXkvMSJ9LHsiYnVja2V0IjoiZGFzcHJvZC1zdG9yZSJ9LHsic3VjY2Vzc19hY3Rpb25fc3RhdHVzIjoiMjAwIn0sWyJzdGFydHMtd2l0aCIsIiRzdWNjZXNzX2FjdGlvbl9yZWRpcmVjdCIsIiJdLFsic3RhcnRzLXdpdGgiLCIkY29udGVudC1UeXBlIiwiYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtIl0seyJ4LWFtei1zZXJ2ZXItc2lkZS1lbmNyeXB0aW9uIjoiQUVTMjU2In0sWyJjb250ZW50LWxlbmd0aC1yYW5nZSIsIjAiLCIxMDQ4NTc2MDAiXSx7IngtYW16LWNyZWRlbnRpYWwiOiJBU0lBVEdWSlpLTTNQNjRNWVRVVC8yMDE5MDcwNC91cy1lYXN0LTEvczMvYXdzNF9yZXF1ZXN0LyJ9LHsieC1hbXotYWxnb3JpdGhtIjoiQVdTNC1ITUFDLVNIQTI1NiJ9LHsieC1hbXotZGF0ZSI6IjIwMTkwNzA0VDA2MTY1MloifSx7IngtYW16LXNlY3VyaXR5LXRva2VuIjoiQWdvSmIzSnBaMmx1WDJWakVKNy8vLy8vLy8vLy93RWFDWFZ6TFdWaGMzUXRNU0pHTUVRQ0lDRFJzaTZqTVdSTVBodmpnZ2VISS9GS0FPeklCUkpWeTBjK3E4LzVhOVEwQWlCT29HVUFCTTJFSXRmWUNRbWhYTlNHOEl0NWJhVjNqNmNWOGVydXZHaWEweXFiQWdqbi8vLy8vLy8vLy84QkVBRWFEREl5TURRM016RTFNak14TUNJTWRNZ0hTeWdVZHNIMWxOQjZLdThCbDQ0a2YybTFMVzB3UDZMdGdiYmkxbzdjb1dKdnV6b0Mrb09NYit0ZlcrdysxdVZrWlJHVVZ3cnlwRERrZjZKSDU5dTNIQmYrc3VXcEQwaWRGcnh2emxJNnNMeXlTbDAramJkekhWWHF0cW1jYUdTQzhHdzZ3amptbnZkRUpoWFljWlE2cml0bURyRXFJT3BqcDJqL1RxcFpsYkgxOFpJQnlHZCtyU1JXV3FTaHBLLzk3VWtjNStHS2xtRlpMU1cxVzR4RTJLZWhZU0tmNlIzdTZFYXUxSjNvNVpad21IQTlPNFAxc294YnlJcExrZUoxVlBCU1RleFBwSTZEWnZnNzk3cDgxUG1SQ0FsaW1Gd2NPZWJXYVV5a203RnNVTE40YUMwMUcrS1RESG9CQi91enRYNUlVK0FZVnBWcFNMc3duYVgyNkFVNnRRSFJEblFoNzRBMW5Cd3lxS0VKMWhSOW8wSVFuQjAvclhvbVJ5Q291QXZHaEpxSzF5NnNhWitObjVxTWdjTEhrSWR1NW83UzBrV3lDMldWR01lZlZkK0pSY05zaHdadGxNNGtSK3lkU2VXS2dSaWQyMGEvMWVuRHU3R0RaVEkrRXFsdVk1a09RdFRoWFdYWm91SlhIa3MrNzVuWnF1NCtCbmQ3bkt2TGVlZGNTZ05TNTJaTzZ4UjQ5N3dzK1JhT2F3ZENDamxoM1krUkdhS0lkQlhBcFFldUxEY3V0UGF3VkVQR3N3dlpJWnUySEdnbXI1NnQifV19 \
  --form success_action_status=200 \
  --form success_action_redirect= \
  --form x-amz-signature=812f298397cd20d872839ff68b1aed2e259ee5203aea436c67bc1dac18b15b98 \
  --form x-amz-credential=ASIATGVJZKM3P64MYTUT/20190704/us-east-1/s3/aws4_request/ \
  --form x-amz-algorithm=AWS4-HMAC-SHA256 \
  --form x-amz-date=20190704T061652Z \
  --form x-amz-server-side-encryption=AES256 \
  --form 'x-amz-security-token=AgoJb3JpZ2luX2VjEJ7//////////wEaCXVzLWVhc3QtMSJGMEQCICDRsi6jMWRMPhvjggeHI/FKAOzIBRJVy0c+q8/5a9Q0AiBOoGUABM2EItfYCQmhXNSG8It5baV3j6cV8eruvGia0yqbAgjn//////////8BEAEaDDIyMDQ3MzE1MjMxMCIMdMgHSygUdsH1lNB6Ku8Bl44kf2m1LW0wP6Ltgbbi1o7coWJvuzoC+oOMb+tfW+w+1uVkZRGUVwrypDDkf6JH59u3HBf+suWpD0idFrxvzlI6sLyySl0+jbdzHVXqtqmcaGSC8Gw6wjjmnvdEJhXYcZQ6ritmDrEqIOpjp2j/TqpZlbH18ZIByGd+rSRWWqShpK/97Ukc5+GKlmFZLSW1W4xE2KehYSKf6R3u6Eau1J3o5ZZwmHA9O4P1soxbyIpLkeJ1VPBSTexPpI6DZvg797p81PmRCAlimFwcOebWaUykm7FsULN4aC01G+KTDHoBB/uztX5IU+AYVpVpSLswnaX26AU6tQHRDnQh74A1nBwyqKEJ1hR9o0IQnB0/rXomRyCouAvGhJqK1y6saZ+Nn5qMgcLHkIdu5o7S0kWyC2WVGMefVd+JRcNshwZtlM4kR+ydSeWKgRid20a/1enDu7GDZTI+EqluY5kOQtThXWXZouJXHks+75nZqu4+Bnd7nKvLeedcSgNS52ZO6xR497ws+RaOawdCCjlh3Y+RGaKIdBXApQeuLDcutPawVEPGswvZIZu2HGgmr56t' \
  --form file=D:\Temp\UpdateParameters.zip \
```

### Step7:AppBundle

This API endpoint is useful to test, if our addin package is successfully uploaded or not.
From the current response, extract `$.package` place it in the any browser or just click on hyperlink in insomania.

A typical `$.package` url is 

<https://dasprod-store.s3.amazonaws.com/apps/uLIUN5bLOzQCmoEARUNfG0g1Nr66hj7S/ForgeTestBundleToday/1?AWSAccessKeyId=ASIATGVJZKM3P64MYTUT&Expires=1562224843&x-amz-security-token=AgoJb3JpZ2luX2VjEJ7%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCICDRsi6jMWRMPhvjggeHI%2FFKAOzIBRJVy0c%2Bq8%2F5a9Q0AiBOoGUABM2EItfYCQmhXNSG8It5baV3j6cV8eruvGia0yqbAgjn%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDIyMDQ3MzE1MjMxMCIMdMgHSygUdsH1lNB6Ku8Bl44kf2m1LW0wP6Ltgbbi1o7coWJvuzoC%2BoOMb%2BtfW%2Bw%2B1uVkZRGUVwrypDDkf6JH59u3HBf%2BsuWpD0idFrxvzlI6sLyySl0%2BjbdzHVXqtqmcaGSC8Gw6wjjmnvdEJhXYcZQ6ritmDrEqIOpjp2j%2FTqpZlbH18ZIByGd%2BrSRWWqShpK%2F97Ukc5%2BGKlmFZLSW1W4xE2KehYSKf6R3u6Eau1J3o5ZZwmHA9O4P1soxbyIpLkeJ1VPBSTexPpI6DZvg797p81PmRCAlimFwcOebWaUykm7FsULN4aC01G%2BKTDHoBB%2FuztX5IU%2BAYVpVpSLswnaX26AU6tQHRDnQh74A1nBwyqKEJ1hR9o0IQnB0%2FrXomRyCouAvGhJqK1y6saZ%2BNn5qMgcLHkIdu5o7S0kWyC2WVGMefVd%2BJRcNshwZtlM4kR%2BydSeWKgRid20a%2F1enDu7GDZTI%2BEqluY5kOQtThXWXZouJXHks%2B75nZqu4%2BBnd7nKvLeedcSgNS52ZO6xR497ws%2BRaOawdCCjlh3Y%2BRGaKIdBXApQeuLDcutPawVEPGswvZIZu2HGgmr56t&Signature=f17Tq31%2Bvo4Tnbd%2FQKOvc5pZQ5w%3D>




The expected behavior is it browser should open download explorer window, after selection download location on Windows explorer, name the file as desired with `.zip` extension.

### Step 8: Create Workitem

This request to the Design Automation V3 API will launch the execution of the activity that was created during the request `Create Activity`


```
"onComplete": {
			"verb": "post",
			"url": "https://webhook.site/278c184d-7fa9-4997-b5b4-a6074b1372cc"
		}
```
In create workitem payload, there is an optional argument `onComplete` which is a webhook supplied by design automation, this upon registering, once the workitem job is completed following information is posted to the webhook registered url.

```
"onComplete": {
    "ondemand": true,
    "optional": true,
    "url": "https://webhook.site/278c184d-7fa9-4997-b5b4-a6074b1372cc",
    "verb": "post"
  }  
```
### Step 9: GetWorkItem

To query the status of workitem


### Troubleshooting:

For any reason if you are facing issues with bundle or importing the insomania json.

Please go through below link, I have recorded the steps using Forge With Insomania,

https://autode.sk/2Yvej2I

### Insomnia Env variables

![](ForgeEnv-FDA.JPG)