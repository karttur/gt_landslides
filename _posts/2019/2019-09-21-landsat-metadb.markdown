---
layout: post
title: Translate Landsat meta data
categories: blog
excerpt: "Define meta data database columns and translate to different meta data sources"
tags:
  - GeoImagine
  - Landsat templates
  - meta data
image: avg-trmm-3b43v7-precip_3B43_trmm_2001-2016_A
date: '2019-09-21 08:51'
modified: '2019-09-21 08:51'
comments: true
share: true

---
<script src="https://karttur.github.io/common/assets/js/karttur/togglediv.js"></script>

Before processing any Landsat data, you must add a translator for the different kinds of meta data you can encounter. The translator converts  meta data formats to the standardized format used by Karttur's GeoImagine Framework

## Introduction

The Landsat series of platforms and sensors goes back to 1972 when Landsat 1 was launched. Since then a range of different software, versions and principles have been applied for saving and storing meta data. There are literarily hundreds of different formats. Karttur´s GeoImagine Framework can be tuned to identify most of the present and historical Landsat meta data you are most likely to encounter. As the USGS online Landsat library has grown over the past years, it is, however, almost always better to retrieve the Landsat data, with the USGS standardised meta data formats, from them.

## Prerequisites

To follow the steps below you must have setup Karttur's GeoImagine Framework as described in the blog on [Setup and run Karttur's GeoImagine Framework](https://karttur.github.io/geoimagine/setup/).

## Landsat templates

Recognising and defining Landsat data in Karttur's GeoImagine Framework requires information related to both the Landsat scene and the each individual band. The system is built up to allow any Landsat data to be recognised and defined, including legacy scenes and bands from old archives.

### Landsat scene templates

The scene template contains data related to Landsat scenes, and is used for recognising which platform and sensor a particular scene stems from. It also recognises the different (historical and present) processing any particular scene was constructed from. The scene template can recognise both complete scenes (e.g. in different zip formats, or as multi-band images) as well as which platform, sensor and pre-processing an individual band belongs to.

#### Scene templates xml code

The scene template definition below is for a standard USGS Landsat 5 Thematic Mapper (TM) 5 scene (LT05). The details reveal the product (L1TP), which collection it belongs to (1), the tier or collection category (T1), and the units of the cell data (DN). The subtype is an internal identifier for Karttur's GeoImagine Framework.

```
	<process processid ='landsatscenetemplate' version ='1.3'>
		<parameters source ='LT05' product = 'L1TP' subtype='T' collcat = 'T1' collnr='1'
		proccat = 'DN'
		scenenameform = 'LT05_L1TP_ppprrr_YYYYMMDD_yyyymmdd_01_T1.tar.gz'
		basenameform = 'LT05_L1TP_ppprrr_YYYYMMDD_yyyymmdd_01_T1'
		scenecompstr = 'LT05_L1TP_??????_????????_????????_01_T1.tar.gz'
		basecompstr = 'LT05_L1TP_??????_????????_????????_01_T1'
		archive = 'USGS'		
		>
		</parameters>
	</process>
```

#### Complete scene xml file

The complete xml file (<span class='file'>templatescenes_2019.xml</span>) include templates for most Landsat TM, Enhanced TM (ETM), Operational Land Imager (OLI) and Thermal Infrared Sensor (TIRS) scenes available via USGS.

{% capture foo %}{{page.templatescenes_2019}}{% endcapture %}
{% include xml/templatescenes_2019.html foo=foo %}

#### Running the process _landsatscenetemplate_

To insert the templates defined in the file <span class='file'>templatescenes_2019.xml</span> in your db, you have to call it from within the Framework. Either you call it directly, or you call a text <span class='file'>.txt</span> file that in turn lists the xml file.

The example below shows how to use a text file (<span class='file'>landsat.txt</span>).

```
#from . import ReadXMLProcesses

if __name__ == "__main__":

    verbose = 1

    projFN = '/full/path/to/landsat.txt'
    projFN = 'relative/path/to/landsat.txt'

    procLL = ReadXMLProcesses(projFN,verbose)

    RunProcesses(procLL,verbose)
```

And then add the xml file defining the Landsat scenes in the text file (<span class='file'>landsat.txt</span>):

```
# Insert Landsat template scenes in db
templatescenes_2019.xml
```

### Landsat band templates

Defining a Landsat band requires both recognising the band, defining its content and data format and then also information related to how the band should be processed and named. Apart from the bands, most scenes also comes with additional, spatial support data as well as non-spatial data. Additional support data include quality and masks for clouds, cloud shadows, snow, ice and water. Non-spatial data include information on scene and band quality, geometric correction, processing etc. In Karttur's GeoImagine Framework the definition of templates for bands, other spatial data and non-spatial data (i.e. meta data) differ slightly and thus also the templates differ slightly.

#### Band templates xml code

The example below shows the definition of the Thematic Mapper (TM) sensor band 1 (blue wavelength) for Landsat 5, pre-processed to level L1TP and quality tagged as tier T2 in collection 1. Tier and collection are USGS defined categories related to quality and processing. The Framework process for inserting band template data to the db is _landsatbandtemplate_.

```
<process processid ='landsatbandtemplate' version ='1.3'>
		<parameters
      source ='LT05'
      product = 'L1TP'
      subtype='T'   
      collcat = 'T2'
      collnr='1'
			proccat = 'DN'   
			scalefac = '1'
			offsetadd='0'
			band = 'bl'
			bandnr = '1'
			bandcat = 'reflectance'
			prefix = 'bl-dn'
			suffix = '0'
			pattern = '_B1.TIF'
			measure = 'I'
			dataunit = 'dn'
			celltype ='Byte'
			cellnull = '0'
			folder = 'dnbands'
			fileext = 'tif'
			required = 'Y'
		>
		</parameters>
</process>
```

All bands for Landsat TM (flown on Landsat 4 and 5), Enhanced TM (ETM) (flown on Landsat 7), Operational Land Imager (OLI) and Thermal Infrared Sensor) (both the latter flown on Landsat 8), as available from USGS??? are included in the [project xml file repository](#???).

It is in principle possible to define templates for any Landsat bands, including data downloaded and processed by different receiving stations since the start of Landsat program.

#### Support template xml

The example below belongs to the same scene as the band above and defines the quality support data. The Framework process for inserting support template data to the db is _landsatsupporttemplate_.

```
	<process processid ='landsatsupporttemplate' version ='1.3'>
		<overwrite>N</overwrite>
		<delete>N</delete>
		<parameters source ='LT05' product = 'L1TP' subtype='T' collcat = 'T2' collnr='1'
			proccat = 'DN'
			scalefac = '1'
			offsetadd='0'
			band = 'bqa'
			prefix = 'bqa'
			suffix = '0'
			pattern = '_BQA.TIF'
			measure = 'N'
			dataunit = 'dn'
			celltype ='Int16'
			cellnull = '1'
			folder = 'dndqa'
			fileext = 'tif'
			required = 'Y'
		>
		</parameters>
	</process>
```

#### Meta data template xml

The example below show meta data for to the same scene as the band and support above. The Framework process for inserting meta data templates to the db is _landsatmetatemplate_.

```
	<process processid ='landsatmetatemplate' version ='1.3'>
		<overwrite>N</overwrite>
		<delete>N</delete>
		<parameters source ='LT05' product = 'L1TP' subtype='T' collcat = 'T2' collnr='1'
			proccat = 'DN'
			band = 'mtl'
			prefix = 'mtl'
			suffix = '0'
			pattern = 'MTL.txt'
			measure = 'N'
			folder = 'dnmeta'
			fileext = 'txt'
			required = 'Y'
		>
		</parameters>
	</process>
```

#### Complete band, support and meta data xml file

The complete xml file (<span class='file'>templatelayers_LT05_L1TP_T_T2_01_2019}.xml</span>) for the bands, support layers and meta data for the scene (Landsat TM 5) is under the button below. The xml file also, together with xml files defining a range of other scene bands, is available in the online repo.

{% capture foo %}{{page.templatelayers_LT05_L1TP_T_T2_01_2019}}{% endcapture %}
{% include xml/templatelayers_LT05_L1TP_T_T2_01_2019.html foo=foo %}

#### Running the processes

With the xml encoded templates defined as shown above, all that is needed for adding them to the db is to add the file names to the text file used above for linking the xml file defining the scenes:

And then add the xml file defining the Landsat scenes in the text file (<span class='file'>landsat.txt</span>):

```
# Insert Landsat template scenes to db
templatescenes_2019.xml

# Insert band, support and meta data to db
templatelayers_LT05_L1TP_T_T1_01_2019.xml
templatelayers_LT05_L1TP_T_T2_01_2019.xml
templatelayers_LE07_L1TP_T_T1_01_2019.xml
templatelayers_LE07_L1TP_T_T2_01_2019.xml
templatelayers_LC08_L1TP_T_T1_01_2019.xml
templatelayers_LT05_L1GS_T_T1_01_2019.xml
templatelayers_LC08_L1GT_T_T2_01_2019.xml
templatelayers_LE07_L1GT_T_T2_01_2019.xml
templatelayers_LE07_L1TP_T_RT_01_2019.xml
```
