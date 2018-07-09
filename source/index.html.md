---
title: ISIC Archive API Documentation
language_tabs:
  - python: Python
  - R: R
toc_footers:
  - >-
    <a href="https://mermade.github.io/shins/asyncapi.html">See AsyncAPI
    example</a>
includes: []
search: true
highlight_theme: darkula
headingLevel: 2

---

<h1 id="Swagger-Petstore">ISIC Archive API Documentation</h1>

> Scroll down for code samples, example requests and responses. Select a language for code samples from the tabs above or the mobile navigation menu.

Welcome to the ISIC Archive API! You can use our API to access numerous endpoints, which will allow you to query our database to access images and associated data.

We have language bindings in Python and R! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs at the top.

<b>A full list of API endpoints can be found at: [https://isic-archive.com/api/v1](https://isic-archive.com/api/v1)</b>


#Libraries
Libraries used in the code samples can be found at the links below:
* Python: https://raw.githubusercontent.com/ImageMarkup/isic-archive/master/scripts/isic_api.py
* R: https://raw.githubusercontent.com/ImageMarkup/isic-archive/master/scripts/isic_api.R

Notes: 
* For the Python module: If using the public API without a username and password, use <b>ISICApi()</b> instead of ISICApi(username='', password='')


# Authentication

- HTTP Basic Auth 

    - Authorization Endpoint: https://isic-archive.com/api/v1/user/authentication

|Argument|Description|
|---|---|
|username|ISIC Archive Username|
|password|ISIC Archive Password|

> Code Sample

```python
from isic_api import ISICApi

#Insert Username and Password Below
api = ISICApi(username="ISIC", password="ARCHIVE")
```

```R
library(httr)
library(jsonlite)
library(gtools)

login_ = function(username, password) {
  isicauth = httr::GET(url="https://isic-archive.com/api/v1/user/authentication", config=c(authenticate(user=username, password = password)),verbose())
  logindata = fromJSON(rawToChar(isicauth$content))
  return(logindata)
}

get_data = function(desired_endpoint_url, logindata) {
  data = httr::GET(url=desired_endpoint_url, config=c(progress("down"), accept_json(), add_headers("Girder-Token" = logindata$authToken$token)))
  data = fromJSON(rawToChar(data$content))
  return(data)
}

username="" # Insert ISIC Archive Username
password = .rs.askForPassword("password") #this funtion is specific to R-Studio. If not using R-Studio then just replace with password as string
logindata = login_(username, password)
remove(password)
```
# Images

## Get List of Images
* Endpoints used: 
	* https://isic-archive.com/api/v1/image

|Argument|Description|Example Value|
|---|---|---|
|limit|Limit on records returned|50 (default:50)|
|offset|Offset for records returned (i.e. for pagination)|50 (default: 0)|
|sort|Field for sorting records returned| name (default: name)|
|name|Find image with specific name| ISIC_0000000 (default: empty)|
|imageIds|A JSON array of up to 100 image IDs, which will be exclusively included|5436e3abbae478396759f0cf (default: empty)|

> Code Sample

```python
#Insert Username and Password Below
api = ISICApi(username="ISIC", password="ARCHIVE")
imageList = api.getJson('image?limit=100&offset=0&sort=name')
```

```R
desired_endpoint = "https://isic-archive.com/api/v1/image?limit=100&offset=0&sort=name"
imageList = get_data(desired_endpoint, logindata)
```
## Get Details/Metadata for Images
* Endpoints used:
	* https://isic-archive.com/api/v1/image
	* https://isic-archive.com/api/v1/image/{image_id}

|Argument|Description|Example Value|
|---|---|---|
|image_id|The '_id' field of records returned from the /image endpoint|5436e3abbae478396759f0cf (default:empty)|

> Code Sample
```python
# Initialize the API; no login is necessary for public data
api = ISICApi(username='ISIC', password='ARCHIVE')
outputFileName = 'imagedata'

imageList = api.getJson('image?limit=100&offset=0&sort=name')

print('Fetching metadata for %s images' % len(imageList))
imageDetails = []
for image in imageList:
    print(' ', image['name'])
    # Fetch the full image details
    imageDetail = api.getJson('image/%s' % image['_id'])
    imageDetails.append(imageDetail)

# Determine the union of all image metadata fields
metadataFields = set(
    field
    for imageDetail in imageDetails
    for field in imageDetail['meta']['clinical'].viewkeys()
)
metadataFields = ['isic_id'] + sorted(metadataFields)

# Write the metadata to a CSV
print('Writing metadata to CSV: %s' % outputFileName+'.csv')
with open(outputFilePath+'.csv', 'wb') as outputStream:
    csvWriter = csv.DictWriter(outputStream, metadataFields)
    csvWriter.writeheader()
    for imageDetail in imageDetails:
        rowDict = imageDetail['meta']['clinical'].copy()
        rowDict['isic_id'] = imageDetail['name']
        csvWriter.writerow(rowDict)
```

```R
desired_endpoint = "https://isic-archive.com/api/v1/image?limit=100&offset=0&sort=name&sortdir=1"
imageList = get_data(desired_endpoint, logindata)

imageMetadata = data.frame()
for (i in 1:length(imageList$`_id`)) {
  desired_endpoint = paste0("https://isic-archive.com/api/v1/image/", imageList$`_id`[i])
  data = get_data(desired_endpoint, logindata)
  imageMetadata = smartbind(imageMetadata, unlist(data))
}

write.csv(imageMetadata, 'imageMetadata.csv', row.names=FALSE)
```

## Download Images
* Endpoints used:
	* https://isic-archive.com/api/v1/image
	* https://isic-archive.com/api/v1/image/{image_id}/download

|Argument|Description|Example Value|
|---|---|---|
|image_id|The '_id' field of records returned from the /image endpoint|5436e3abbae478396759f0cf (default:empty)|

Notes:
* Replace /download with /thumbnail to download thumbnails instead of full-size images

> Code Sample
```python
import urllib
import os

# Initialize the API; no login is necessary for public data
api = ISICApi()
savePath = 'ISICArchive/'

if not os.path.exists(savePath):
    os.makedirs(savePath)

imageList = api.getJson('image?limit=100&offset=0&sort=name')

print('Downloading %s images' % len(imageList))
imageDetails = []
for image in imageList:
    print(image['_id'])
    imageFileResp = api.get('image/%s/download' % image['_id'])
    imageFileResp.raise_for_status()
    imageFileOutputPath = os.path.join(savePath, '%s.jpg' % image['name'])
    with open(imageFileOutputPath, 'wb') as imageFileOutputStream:
        for chunk in imageFileResp:
            imageFileOutputStream.write(chunk)
```

```R
desired_endpoint = "https://isic-archive.com/api/v1/image?limit=100&offset=0&sort=name&sortdir=1"
imageList = get_data(desired_endpoint, logindata)

savePath = "ISICArchive/"

if (!dir.exists(savePath)){
  dir.create(savePath)
}

for (i in 1:length(imageList$`_id`)) {
  getURL = paste0("https://isic-archive.com/api/v1/image/", imageList$`_id`[i], "/download")
  saveFile = paste0(savePath, imageList$`_id`[i], ".jpg")
  download.file(url=getURL, destfile = saveFile, quiet = TRUE, method="auto", mode="wb")
}
```


# Studies
## Get List of Studies
* Endpoints used: 
	* https://isic-archive.com/api/v1/study

|Argument|Description|Example Value|
|---|---|---|
|limit|Limit on records returned|50 (default:50)|
|offset|Offset for records returned (i.e. for pagination)|50 (default: 0)|
|state|Filter for study status, either 'active' or 'complete'| complete (default: empty)|
|userId|Filter studies to those containing a particular user ID, or 'me'|me (default: empty)|

> Code Sample

```python
#Insert Username and Password Below
api = ISICApi(username="ISIC", password="ARCHIVE")
studyList = api.getJson('study?limit=50&state=complete')
```

```R
desired_endpoint = "https://isic-archive.com/api/v1/study?limit=10"
studyList = get_data(desired_endpoint, logindata)
```

## Get Details for Study
* Endpoints used:
	* https://isic-archive.com/api/v1/study
	* https://isic-archive.com/api/v1/study/{study_id}

|Argument|Description|Example Value|
|---|---|---|
|study_id|The '_id' field of records returned from the /study endpoint|573f11119fc3c132505c0ee7 (default: empty)|
|format|Desired format for data output, can be 'json' or 'csv'|json (default: json)|

Notes:
* This endpoint returns the study ID, date created, study description (if available), and list of images. The list of images included in a study is usually the data of interest from this endpoint


> Code Sample
```python
import pandas as pd

# Initialize the API; no login is necessary for public data
api = ISICApi(username='ISIC', password='ARCHIVE')
outputFileName = 'imagedata'

studyData = api.getJson('study/573f11119fc3c132505c0ee7')
studyData_images = pd.DataFrame(studyData['images'])

studyData_images.to_csv('studyDataImages.csv')

```

```R
desired_endpoint = "https://isic-archive.com/api/v1/study/573f11119fc3c132505c0ee7"
studyData = get_data(desired_endpoint, logindata)

studyData_images = data.frame(studyData$images, stringsAsFactors=FALSE)

write.csv(studyData_images, 'studyDataImages.csv', row.names=FALSE)
```








# Superpixels

## Download Superpixel Masks
* Endpoints used:
	* https://isic-archive.com/api/v1/image
	* https://isic-archive.com/api/v1/image/{image_id}/superpixels

|Argument|Description|Example Value|
|---|---|---|
|image_id|The '_id' field of records returned from the /image endpoint|5436e3abbae478396759f0cf (default:empty)|

> Code Sample
```python
import urllib
import os

# Initialize the API; no login is necessary for public data
api = ISICApi()
savePath = 'ISICArchive/'

if not os.path.exists(savePath):
    os.makedirs(savePath)

imageList = api.getJson('image?limit=100&offset=0&sort=name')

print('Downloading %s images' % len(imageList))

for image in imageList:
    print(image['_id'])
    imageFileResp = api.get('image/%s/superpixels' % image['_id'])
    imageFileResp.raise_for_status()
    imageFileOutputPath = os.path.join(savePath, '%s_superpixels.jpg' % image['name'])
    with open(imageFileOutputPath, 'wb') as imageFileOutputStream:
        for chunk in imageFileResp:
            imageFileOutputStream.write(chunk)
```

```R
desired_endpoint = "https://isic-archive.com/api/v1/image?limit=100&offset=0&sort=name&sortdir=1"
imageList = get_data(desired_endpoint, logindata)

savePath = "ISICArchive/"

if (!dir.exists(savePath)){
  dir.create(savePath)
}

for (i in 1:length(imageList$`_id`)) {
  getURL = paste0("https://isic-archive.com/api/v1/image/", imageList$`_id`[i], "/superpixels")
  saveFile = paste0(savePath, imageList$`_id`[i], "_superpixels.jpg")
  download.file(url=getURL, destfile = saveFile, quiet = TRUE, method="auto", mode="wb")
}
```

# Segmentations

## Get List of Image Segmentations
* Endpoints used: 
	* https://isic-archive.com/api/v1/segmentation

|Argument|Description|Example Value|
|---|---|---|
|image_id|The '_id' field of the image for which available segmentations are being requested|5436e3abbae478396759f0cf (default:empty)|


> Code Sample

```python
#Insert Username and Password Below
api = ISICApi(username="ISIC", password="ARCHIVE")
imageId = '5436e3abbae478396759f0cf' #Using image ISIC_0000000 as example
imageSegmentationData = api.getJson('segmentation?imageId='+imageId)
```

```R
imageId = '5436e3abbae478396759f0cf' #Using image ISIC_0000000 as example
desired_endpoint = paste0("https://isic-archive.com/api/v1/segmentation?imageId=", imageId)
imageSegmentationData = get_data(desired_endpoint, logindata)
```

## Get Details for a Segmentation
* Endpoints used: 
	* https://isic-archive.com/api/v1/segmentation/{segmentation_id}

|Argument|Description|Example Value|
|---|---|---|
|segmentation_id|The '_id' field of the record from the /segmentation endpoint |5463934bbae47821f88025ad (default:empty)|


> Code Sample

```python
#Insert Username and Password Below
api = ISICApi(username="ISIC", password="ARCHIVE")
segmentationId = '5463934bbae47821f88025ad' #Using image ISIC_0000000 as example
segmentationData = api.getJson('segmentation/'+segmentationId)
```

```R
segmentationId = '5463934bbae47821f88025ad' #Using image ISIC_0000000 as example
desired_endpoint = paste0("https://isic-archive.com/api/v1/segmentation/", segmentationId)
segmentationData = get_data(desired_endpoint, logindata)
```

## Download Segmentation Mask
* Endpoints used:
	* https://isic-archive.com/api/v1/segmentation
	* https://isic-archive.com/api/v1/segmentation/{segmentation_id}/mask

|Argument|Description|Example Value|
|---|---|---|
|segmentation_id|The '_id' field of the record from the /segmentation endpoint |5463934bbae47821f88025ad (default:empty)|

> Code Sample
```python
import urllib
import os

# Initialize the API; no login is necessary for public data
api = ISICApi()
savePath = 'ISICArchive/'

if not os.path.exists(savePath):
    os.makedirs(savePath)

imageId = '5436e3abbae478396759f0cf'
segmentationList = api.getJson('segmentation?imageId='+imageId)

print('Downloading %s images' % len(segmentationList))

for segmentation in segmentationList:
    print(segmentation['_id'])
    imageFileResp = api.get('segmentation/%s/mask' % segmentation['_id'])
    imageFileResp.raise_for_status()
    imageFileOutputPath = os.path.join(savePath, '%s_segmentationMask.jpg' % segmentation['name'])
    with open(imageFileOutputPath, 'wb') as imageFileOutputStream:
        for chunk in imageFileResp:
            imageFileOutputStream.write(chunk)
```

```R
imageId = '5436e3abbae478396759f0cf'
desired_endpoint = paste0("https://isic-archive.com/api/v1/segmentation?imageId=", imageId)
segmentationList = get_data(desired_endpoint, logindata)

savePath = "ISICArchive/"

if (!dir.exists(savePath)){
  dir.create(savePath)
}

for (i in 1:length(segmentationList$`_id`)) {
  getURL = paste0("https://isic-archive.com/api/v1/segmentation/", segmentationList$`_id`[i], "/mask")
  saveFile = paste0(savePath, segmentationList$`_id`[i], "_segmentationMask.jpg")
  download.file(url=getURL, destfile = saveFile, quiet = TRUE, method="auto", mode="wb")
}
```



# Annotations



<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "WebAPI",
  "description": ":dog: :cat: :rabbit: This is a sample server Petstore server.  You can find out more about Swagger at [http://swagger.io](http://swagger.io) or on [irc.freenode.net, #swagger](http://swagger.io/irc/).  For this sample, you can use the api key `special-key` to test the authorization filters.",
  "documentation": "https://mermade.github.io/shins/asyncapi.html",
  "termsOfService": "http://swagger.io/terms/",
  
  "name": "Swagger Petstore"
}
</script>

