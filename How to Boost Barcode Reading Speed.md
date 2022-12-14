----
## Contents
### 2022.11.25
Topic1. DBR algorithm process flow & parameter design<br />
* [DBR algorithm architecture](https://www.dynamsoft.com/barcode-reader/docs/core/introduction/architecture.html?ver=latest)
* [DBR parameter template structure](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/structure-and-interfaces-of-parameters.html?ver=latest)

Topic2. Speed-up-parameters
* [1.BinarizationModes](#233-adapt-image-binarization-for-speed)<br />
* [2.TextureDetectionModes](#221-filter-texture)<br />
* [3.TextFilterModes](#222-filter-text)
 
Topic3. Online Demo & Experiment<br />
* [Online Demo](https://demo.dynamsoft.com/barcode-reader/)

----

# How to Boost Barcode Reading Speed
There are 3 basic metrics for measuring the performance of a barcode reader application: speed, accuracy, and read rate. The Dynamsoft Barcode Reader SDK (DBR) has been carefully designed to offer excellent performance in all three of these metrics. In this article, we investigate all the possible ways in which you can configure DBR to focus on speed.<br />This guide explores how DBR can be used to its full-speed potential, and it will be divided into three sections for three types of methods: 

- The first section addresses some of the common and simple methods you can use to effectively increase speed metrics.
- The second section delves into the various stages of DBR's algorithm and how we can improve the speed at each of the crucial stages.
- The third and last section will cover some of the much less popular and used methods that our SDK can use to potentially increase the speed metric should the other methods not be sufficient.

![](https://cdn.nlark.com/yuque/__latex/be927f177f815e444e5a1f134bc15ad6.svg#card=math&code=Accuracy%20%3D%20%5Cfrac%7BNumber~of~Correctly~Decoded~Barcode~Results%7D%7BNumber~of~All~Decoded~Barcode~Results%7D&id=qiuos)(read correctly)<br />![](https://cdn.nlark.com/yuque/__latex/9892190912d89445b8e6a7da36858a16.svg#card=math&code=Speed%20%3D%20%5Cfrac%7BNumber~of~All~Decoded~Barcode~Results%7D%7BTotal~Time~Consumed%7D&id=tK7Wg)???read fast???<br />![](https://cdn.nlark.com/yuque/__latex/fbbe588b00b860326e170ab5f2b5fa97.svg#card=math&code=Read~Rate%20%3D%20%5Cfrac%7BNumber~of~All~Decoded~Barcode~Results%7D%7BNumber~of~All~Target~Result%7D&id=khnAT)???read as many as possible???
> Do bear in mind that, if one of these metrics is prioritized, the other two may not be ideal.


---

> Note??? 
> This guide provides sample images and an online experiments, to explore how DBR works better in speed. Readers of this chapter suppose to know:
> - The basic concepts of  the DBR algorithm and process flow.
> - How to set parameters.

DBR's default settings are balanced and powerful, which means it makes a lot of effort when decoding. This runs counter to the speed we are pursuing in this article, we had better cut out all unnecessary parts. 
<a name="lJC3U"></a>
#### 1. Explore the common methods for better speed
<a name="EEzwb"></a>
##### 1.1 Focus on the barcode types of interest
This is probably the most natural setting to start with. By clearly telling DBR what it is looking for, it can quickly skip other types of barcodes that can potentially be on the same image or frame.<br />The related parameters are [BarcodeFormatIds](https://www.dynamsoft.com/barcode-reader/parameters/reference/barcode-format-ids.html?ver=latest) and [BarcodeFormatIds_2](https://www.dynamsoft.com/barcode-reader/parameters/reference/barcode-format-ids-2.html?ver=latest). The former parameter includes the most common barcode types and the latter includes the few unusual types that the SDK supports.

| Parameter | Default settings |
| --- | --- |
| [BarcodeFormatIds](https://www.dynamsoft.com/barcode-reader/parameters/reference/barcode-format-ids.html?ver=latest) | BF_ALL |
| [BarcodeFormatIds_2](https://www.dynamsoft.com/barcode-reader/parameters/reference/barcode-format-ids-2.html?ver=latest) | BF2_NULL |

The complete list of  BarcodeFormatIds and BarcodeFormatIds_2 can be found [here](https://www.dynamsoft.com/barcode-reader/docs/server/programming/python/api-reference/enumeration/format-enums.html?ver=latest#barcodeformat). Please always specify the type(s) of the barcodes you are trying to read.
<a name="dmcRg"></a>
##### **1.2 Set an upper limit to the number of barcodes per image**
By default, DBR tries to find as many barcodes as possible from a given image. Assume the image is very big but has only one barcode at the top, DBR finds the barcode instantly but will spend more time scanning the rest of the image or even try more steps to find more barcodes. By telling DBR that we are only expecting one barcode, it will stop reading the image as soon as that barcode is found.<br />The related parameter is ExpectedBarcodesCount. Note that it can be set to 0 or any natural number. DBR's behavior is as follows:

- ExpectedBarcodesCount is 0: DBR tries to localize barcodes with the first mode set in LocalizationModes. If barcodes are found, the rest of the modes are skipped and the recognition starts right away.
- ExpectedBarcodesCount is > 0: DBR tries to find as many barcodes as defined by this number. If enough barcodes have been found, the rest of the pending operations will be skipped. On the other hand, if the number of found barcodes is less than expected, DBR will exhaust all defined operations to find more until it times out.

| Parameter | Default settings |
| --- | --- |
| [ExpectedBarcodesCount](https://www.dynamsoft.com/barcode-reader/parameters/reference/expected-barcodes-count.html?ver=latest) | 0 |

Do not set the number to exceed the actual number of barcodes on the image. If you are not sure how many barcodes there might be, set ExpectedBarcodesCount to 0. For interactive scenarios, set ExpectedBarcodesCount to 1.
<a name="raQ2n"></a>
##### **1.3 Configure the final decoding process with DeblurModes**
DeblurModes is the last configurable step before a barcode is decoded. By default, DBR disables all deblurring algorithms to decode each localized barcode zone. If your images are of high quality, you can keep the default setting to skip this step, or just try one or two of them. If your image is blurry like the examples below, you may need to turn on DeblurModes:<br />
<!--
<img src="https://cdn.nlark.com/yuque/0/2022/jpeg/22760206/1667350947933-8fee0d88-7962-488b-9ada-b248f3c5bc8f.jpeg" alt="image008" height="20%">
<img src="https://cdn.nlark.com/yuque/0/2022/jpeg/22760206/1667350947950-36e63a28-aca1-4b5a-8e2d-7afb7765d6c0.jpeg" alt="image008" height="20%">
<img src="https://cdn.nlark.com/yuque/0/2022/jpeg/22760206/1667350948072-2edae29a-a18c-4939-bbb6-4a03215c5f03.jpeg" alt="image008" height="20%">
<img src="https://cdn.nlark.com/yuque/0/2022/jpeg/22760206/1667350947960-eb42904e-04bc-402f-b87d-5f003bae76f8.jpeg" alt="image008" height="20%">
-->

| Parameter | Default settings |
| --- | --- |
| [DeblurModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/deblur-modes.html#deblurmodes) | [0,0,0,0,0,0,0,0] |

<a name="T40YJ"></a>
##### **1.4 Optimize frames from a video input**
For interactive barcode reading from video input, you get better speed if:

- the camera that takes the best shot of the intended barcode(s) is used;
- the video frames are clear;
- the video frames are trimmed around the barcode(s) before submitted for barcode reading;
- the video frames are provided in a way that reduces the waiting time of the barcode reading engine.

The [Dynamsoft Camera Enhancer SDK](https://www.dynamsoft.com/camera-enhancer/docs/introduction/) (DCE) is designed to do all of the above like this:<br />NOTE: DCE is available in the SDK packages of DBR iOS v8.2.1+, DBR Android v8.2.1+ and DLR ([Dynamsoft Label Recognizer](https://www.dynamsoft.com/label-recognition/programming/javascript/user-guide.html?ver=latest)) JavaScript v2.2+. Also, the logic is built into the BarcodeScanner class of DBR JavaScript v8.6.0+.

- it comes with camera control and is able to find and open the best-suited camera by default (with support for manual adjustment too);
- it has a lightspeed algorithm to detect whether a frame is blurry. Only clear frames are passed along to DBR (only supported on iOS & Android at present);
- it can crop the video frames with a predefined scan region so that only the relevant part of the frames are passed to DBR;
- it maintains a buffer of frames for DBR to fetch. This reduces the wait time to almost zero.

In addition to the above, DCE also does the following (only supported on iOS & Android at present):

- if a frame is blurred, it tells the camera to adjust its focus;
- if DBR finds the barcode too small in the frame, DCE tells the camera to zoom in.

**Recommendation**

- Take advantage of DCE in interactive barcode reading.
- Set a scan region with DCE and show an indicator of the region so that users know where to focus.
- Try not to use very high resolutions unless absolutely necessary. The default 720P (1280 * 720) usually works fine for most scenarios.

Note that DCE and DBR run in parallel, so it's ok to enable more DCE features without affecting the overall speed.
<a name="N19ec"></a>
#### 2. Delve into the algorithm
![1667179322427.gif](https://cdn.nlark.com/yuque/0/2022/gif/22760206/1667179352163-f8be33a9-15ae-4417-add5-c8b9e77f8c96.gif#clientId=uf9474d8d-bb18-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=467&id=uff909ed4&margin=%5Bobject%20Object%5D&name=1667179322427.gif&originHeight=300&originWidth=300&originalType=binary&ratio=1&rotation=0&showTitle=false&size=173796&status=done&style=none&taskId=u49b90e3d-ae72-4fbc-ab53-9eaefa5b584&title=&width=467)<br />Assume that this is a real case:  a colored 1D code is embedded in the lower left corner of an article.<br />DBR may do these things to make sure that the decoding goes smoothly:

1. convert color image to grayscale image.
2. Filter text, or texture.
3. Enhance image features, including image-preprocessing and binarizing.
4. Localize barcode.
5. Further process(handle difficult barcodes like blurred, Incomplete or deformed)

The strategy is to do only what is necessary and skip the other for the best speed.
<a name="KpDhC"></a>
##### 2.1 Reduce the size of the image
When locating barcodes, DBR scans the whole image, so the larger the size of the image, the more time it takes. To speed things up, we can reduce the size of the original image. Usually,  the reduction is done by scaling down a large image or delimiting the region of interest.

| Parameter | Default settings |
| --- | --- |
| [ScaleDownThreshold](https://www.dynamsoft.com/barcode-reader/parameters/reference/scale-down-threshold.html) | 2300 |
| [RegionDefinition](https://www.dynamsoft.com/barcode-reader/parameters/reference/region-definition/) | [TFM_GENERAL_CONTOUR,0,0,0,0,0,0,0] |
| [RegionPredetectionModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/region-predetection-modes.html#regionpredetectionmodes) | [RPM_GENERAL,0,0,0,0,0,0,0] |

<a name="kxST7"></a>
###### 2.1.1 Scale down a monstrous image
A barcode normally keeps its shape and can be read correctly even when the image gets scaled down. Therefore, DBR shrinks very large images before reading them. The parameter ScaleDownThreshold can be used to determine the threshold beyond which the scale down happens.

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| [ScaleDownThreshold](https://www.dynamsoft.com/barcode-reader/parameters/reference/scale-down-threshold.html) | _int_ | [512, 0x7fffffff] | 2300 |

The default value of ScaleDownThreshold is 2300, try not to set the threshold much lower than the default value as that might make the barcode too dense to be read.
<a name="wHlRO"></a>
###### 2.1.2 Delimit the region of interest
When reading barcodes from a certain type of document or from a video input, the barcode location can usually be predetermined. For example, the barcode on a patient registration form is most likely located in the top 20% of the document, and the barcode that a user is trying to read from a video input is usually located at the center. <br />A group of barcodes appears in the bottom 30 percent of the images:<br />
<!--
<img src="https://cdn.nlark.com/yuque/0/2022/png/22760206/1667353848466-fa48d708-70e2-4d1a-a569-c404f968090a.png" alt="image008" height="20%">
<img src="https://cdn.nlark.com/yuque/0/2022/png/22760206/1667353848448-9433935e-6e94-4a29-bea6-d70e60b7877d.png" alt="image008" height="20%">
<img src="https://cdn.nlark.com/yuque/0/2022/png/22760206/1667353848472-194be3fd-137a-41bf-b6d6-43a3755f8bbb.png" alt="image008" height="25%">
<img src="https://cdn.nlark.com/yuque/0/2022/png/22760206/1667353848526-6421ff95-1a42-402c-bfee-5165dcaa62f3.png" alt="image008" height="25%">
<br />
-->
Recommended Settings:

- RegionTop = 70;
- RegionBottom = 100;
- RegionLeft = 0;
- RegionRight = 100;
- RegionMeasuredByPercentage = 1

The above arguments represent the concept of "the bottom 30 percent in an image".

---

In such cases, we can tell DBR to only read the specific region(s) of interest (ROIs). There are two ways to specify the region:

1. Manually define a region by providing the coordinates of its contours. Each region is defined by a RegionDefinition and then specified by RegionDefinitionNameArray; In runtime settings, the related parameter is Region.
| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| Region/[RegionDefinition](https://www.dynamsoft.com/barcode-reader/parameters/reference/region-definition/) | _N/A_ | _see _[this section](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/region.html?ver=latest#as-json-parameter) | _see _[this section](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/region.html?ver=latest#as-json-parameter) |

When using PublicRuntimeSettings, you can only specify one region, using the JSON template, you can either specify one or more regions. For details, see [How To Read A Specific Area/Region](https://www.dynamsoft.com/barcode-reader/docs/core/programming/features/barcode-scan-region.html?lang=python&&ver=latest).

2. Let DBR find the region based on the colour/grayscale distribution of different parts of the image, this is controlled by the parameter RegionPredetectionModes and it consists of 5 modes, each mode representing a different way to find a region:
- RPM_AUTO: Lets the library choose a mode automatically.
- RPM_GENERAL: Takes the whole image as a region, default setting.
- RPM_GENERAL_RGB_CONTRAST: Detects region using the general algorithm based on RGB colour contrast.
- RPM_GENERAL_GRAY_CONTRAST: Detects region using the general algorithm based on gray contrast.
- RPM_GENERAL_HSV_CONTRAST: Detects region using the general algorithm based on HSV colour contrast.

By default, DBR takes the whole image as a region, using RegionPredetectionModes also requires some computing time, so we are not going to talk about it in this article, you can still read [How To Use Region Predetection](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/scenario-settings/how-to-use-region-predetection.html?ver=latest) if you are interested.<br />It is recommended to set a region manually if it is known that the barcode(s) will be in the general vicinity of the specified region.
<a name="LcEmQ"></a>


##### 2.2 Filter out noises to speed up the localization
| Parameter | Default settings |
| --- | --- |
| [TextureDetectionModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/texture-detection-modes.html#texturedetectionmodes) | [TDM_GENERAL_WIDTH_CONCENTRATION,0,0,0,0,0,0,0] |
| [TextFilterModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/text-filter-modes.html#textfiltermodes) | [TFM_GENERAL_CONTOUR,0,0,0,0,0,0,0] |

The less the noise, the faster the localization. TextureDetectionModes and TextFilterModes can filter the texture and text parts of an image. It is recommended that you turn on both modes to process your image if text and texture exist.
<a name="L7QnU"></a>
###### 2.2.1 Filter texture
In some scenes, the background of images may appear textured, such as a patterned background, screen stripes, etc. As shown below, the barcode background has an odd texture due to the computer screen it is being displayed on. <br />

<img src="asset/TextureDetectionModes/tdm1.JPG" title="On_monitor" width="24%"><img src="asset/TextureDetectionModes/tdm2.JPG" title="On_monitor" width="24%"><img src="asset/TextureDetectionModes/tdm3.JPG" title="On_monitor" width="24%"><img src="asset/TextureDetectionModes/tdm4.JPG" title="On_monitor" width="24%">
<br />
Recommended Settings:
```json
{
    "TextureDetectionModes": [
        {
            "Mode": "TDM_GENERAL_WIDTH_CONCENTRATION",
            "Sensitivity": 5
        }
    ]
}
```
---

Texture may extend the barcode localization time or even lead to localization errors. DBR detects texture by TDM_GENERAL_WIDTH_CONCENTRATION (the only mode currently supported)using the general algorithm. It has the following arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| Sensitivity | _int_ | [1, 9] | 5 |

- A larger Sensitivity value means the library will take more effort to detect texture.

The following two images demonstrate the binarized images used for localization without and with texture detection enabled:<br />![texture.PNG](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667196560410-2c6774d3-c426-4403-b13d-69a04617c332.png#clientId=uf9474d8d-bb18-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=oLwax&margin=%5Bobject%20Object%5D&name=texture.PNG&originHeight=697&originWidth=876&originalType=binary&ratio=1&rotation=0&showTitle=false&size=303083&status=done&style=none&taskId=u4201e874-f20c-424e-b13c-1da9f6b9f81&title=)<br />As you can see, after TextureDetectionModes is enabled, the binarized image looks cleaner and facilitates the subsequent localization.
<a name="ZIVpb"></a>
###### 2.2.2 Filter text
Similarly, TextFilterModes helps to reduce the interference caused by text. <br />

<img src="asset/TextFilterModes/tfm1.png" title="Text_surrounded" width="24%"><img src="asset/TextFilterModes/tfm2.png" title="Text_surrounded" width="24%"><img src="asset/TextFilterModes/tfm3.png" title="Text_surrounded" width="24%"><img src="asset/TextFilterModes/tfm4.png" title="Text_surrounded" width="24%">
<br />
Recommended Settings:
```json
{
    "TextFilterModes": [
        {
            "Mode": "TFM_GENERAL_CONTOUR",
            "Sensitivity": 5
        }
    ]
}
```
---

DBR provides a text filter algorithm based on contour detection called TFM_GENERAL_CONTOUR. In terms of image features, there are some differences between the contour of text and that of barcode, so after contour detection processing, we can use this difference to exclude text regions based on some existing rules. TFM_GENERAL_CONTOUR has the following arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| MinImageDimension | _int_ | [65536, 0x7fffffff] | 65536 |
| Sensitivity | _int_ | [0, 9] | 0 |

- MinImageDimension 65536 (256x256 in pixels) means to enable text filtering when the image size is bigger than this value. The library will enable the region pre-detection feature only when the image dimension is larger than the given value MinImageDimension.
- A larger Sensitivity value means the library will take more effort to filter text. The default value is 0, which means it is set automatically by the library.

For this image, we can set TextFilterModes to TFM_GENERAL_CONTOUR and observe through the intermediate result, IRT_TEXT_ZONE. For easier demonstration purposes, we have merged the result of the IRT_TEXT_ZONE onto the binarized result. As it shown in the following images, the algorithm has recognized almost all the characters in red boxes. After these words are excluded, and we are left with the optimized image.<br /><img src="asset/FilterText.png" title="Filter_text" width="90%"><br />Text and texture contour features are obvious compared with barcode, recommend you enable TextFilterModes and set a proper Sensitivity based on the actual scenario(but do bear in mind that, the process of text filtering will not be performed when using the localization methods LM_CONNECTED_BLOCKS and LM_SCAN_DIRECTLY). Filtering them out in advance reduces the time cost.
<a name="smjmP"></a>
##### 2.3 Accelerate the image conversion
![[23]_[1]_[OriginalImage].png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1666589383517-7d66f7aa-29e6-4c65-ad77-09d85f0593a0.png#clientId=u1dc1a313-f259-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&height=108&id=BPxcA&margin=%5Bobject%20Object%5D&name=%5B23%5D_%5B1%5D_%5BOriginalImage%5D.png&originHeight=240&originWidth=440&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23360&status=error&style=none&taskId=u5c38a09c-a24e-4c04-8bc2-4c4ad94c87d&title=&width=198)<br />![[23]_[2]_[ColourImageConvertedToGrayscale].png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1666589383550-97a23557-29d9-489d-9e5e-95ee8c9b7a27.png#clientId=u1dc1a313-f259-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&height=109&id=YwsXN&margin=%5Bobject%20Object%5D&name=%5B23%5D_%5B2%5D_%5BColourImageConvertedToGrayscale%5D.png&originHeight=240&originWidth=440&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8419&status=error&style=none&taskId=u63b01091-87da-402c-9003-8df3f4cd972&title=&width=200)<br />![3.png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667457879418-aa8a275f-4958-497a-95f5-ac8c3fe9eec8.png#clientId=u6be352d1-062b-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=106&id=u93a95df0&margin=%5Bobject%20Object%5D&name=3.png&originHeight=240&originWidth=440&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10112&status=done&style=none&taskId=u9854ecbc-2d9d-4892-8681-97c10e6099a&title=&width=195)<br />![[23]_[4]_[BinarizedImage].png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1666589383558-6606b04f-d457-4829-a9bd-aa3a5776d775.png#clientId=u1dc1a313-f259-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&height=106&id=E2Qog&margin=%5Bobject%20Object%5D&name=%5B23%5D_%5B4%5D_%5BBinarizedImage%5D.png&originHeight=240&originWidth=440&originalType=binary&ratio=1&rotation=0&showTitle=false&size=930&status=error&style=none&taskId=u3f422e5f-e241-4b10-87b2-03c89ddb242&title=&width=195)<br />Converting the original image to a binarized image makes the barcodes much easier to locate. In most cases, the original image is in color and it first gets converted into a grayscale image, then gets preprocessed, and binarized at last. There is still room for speed improvement in these three steps.

| Parameter | Default settings |
| --- | --- |
| [GrayscaleTransformationModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/grayscale-transformation-modes.html?ver=latest) | [GTM_ORIGINAL,0,0,0,0,0,0,0] |
| [ImagePreprocessingModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/image-preprocessing-modes.html#imagepreprocessingmodes) | [IPM_GENERAL,0,0,0,0,0,0,0] |
| [BinarizationModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/binarization-modes.html#binarizationmodes) | [BM_LOCAL_BLOCK,0,0,0,0,0,0,0] |

<a name="xUVWA"></a>
###### 2.3.1 Select exact operation options
If the original image is not grayscale, DBR will convert it to a grayscale image.  After that, the barcode symbol is either lighter or darker than the background. We call a darker barcode a normal barcode and a lighter barcode an inverted barcode. When locating barcodes, DBR expects the barcodes to be normal. Therefore, if an image in fact has inverted barcodes, DBR needs to invert the color of the image in advance, or this may cause some trouble in the following processes.<br />![inverted.jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/22760206/1667269768054-7dd5cd9e-6642-4f31-ac83-5c6b44b56d4d.jpeg#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=151&id=u8f78d84c&margin=%5Bobject%20Object%5D&name=inverted.jpg&originHeight=84&originWidth=84&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1456&status=done&style=none&taskId=u00ee0886-4fbf-4765-afee-ea071f5b21b&title=original&width=151 "original") <br />![inverted (2).jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/22760206/1667269767990-ed2275ad-3911-4539-a068-38b792e3a663.jpeg#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=152&id=u18f4fb07&margin=%5Bobject%20Object%5D&name=inverted%20%282%29.jpg&originHeight=84&originWidth=84&originalType=binary&ratio=1&rotation=0&showTitle=true&size=2348&status=done&style=none&taskId=u4cbaae8c-8e7d-4bba-ad59-a45f76d1f96&title=inverted&width=152 "inverted")<br />Take the inverted QR Code as an example. If your data is inverted like this, specify GTM_INVERTED, do not specify both.
<a name="cUcgr"></a>
###### 2.3.2 Enhance the grayscale image quality
There are five preprocessing options available in DBR, to enhance the grayscale image quality.

- IPM_GENERAL
- IPM_GRAY_EQUALIZE
- IPM_GRAY_SMOOTH
- IPM_SHARPEN_SMOOTH
- IPM_MORPHOLOGY

Take IPM_SHARPEN_SMOOTH for example, you can see that the barcode edges are more pronounced after the process(In fact, it can be recognized without the IPM_SHARPEN_SMOOTH process as well).<br />
![[23]_[2]_[ColourImageConvertedToGrayscale].png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1666589383550-97a23557-29d9-489d-9e5e-95ee8c9b7a27.png#clientId=u1dc1a313-f259-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&height=178&id=IGN3V&margin=%5Bobject%20Object%5D&name=%5B23%5D_%5B2%5D_%5BColourImageConvertedToGrayscale%5D.png&originHeight=240&originWidth=440&originalType=binary&ratio=1&rotation=0&showTitle=true&size=8419&status=error&style=none&taskId=u63b01091-87da-402c-9003-8df3f4cd972&title=original&width=327 "original")
![[23]_[3]_[PreprocessedImage].png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1666589383552-e8bf91f7-a636-4a3c-853e-d9a087e21d41.png#clientId=u1dc1a313-f259-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&height=175&id=X0gAt&margin=%5Bobject%20Object%5D&name=%5B23%5D_%5B3%5D_%5BPreprocessedImage%5D.png&originHeight=240&originWidth=440&originalType=binary&ratio=1&rotation=0&showTitle=true&size=10112&status=error&style=none&taskId=ue801c42d-9ee2-4e1a-985c-b7a3a8f111e&title=after%20sharpen_smooth&width=320 "after sharpen_smooth")<br />The more image preprocessing modes you specify, the worse speed might get. In most cases, using the default IPM_GENERAL mode is enough.
<a name="Lm9bS"></a>
###### 2.3.3 Adapt image binarization for speed
The goal of image binarization is to enhance the barcode features for easy localization. For this kind of images, binarization is not simple:<br />

<img src="asset/BinarizationModes/bm1.jpg" title="Unbalanced_light" width="24%"><img src="asset/BinarizationModes/bm2.jpg" title="Unbalanced_light" width="24%"><img src="asset/BinarizationModes/bm3.jpg" title="Unbalanced_light" width="24%"><img src="asset/BinarizationModes/bm4.JPG" title="Unbalanced_light" width="24%">
<br />
Recommended Settings:
```json
{
    "BinarizationModes": [
        {
            "Mode": "BM_LOCAL_BLOCK",
            "BlockSizeX": 0,
            "BlockSizeY": 0
        }
    ]
}
```
---

In the binarization process, all the image pixels will be processed into black or white. Depending on the actual lighting conditions, you can choose between BM_THRESHOLD or BM_LOCAL_BLOCK in BinarizationModes.<br />The value of BM_THRESHOLD determines whether a pixel is black or white. Images taken under good lighting conditions can be attempted to use BM_THRESHOLD to complete the binarization process. Otherwise, use BM_LOCAL_BLOCK. <br />Here is an example, suppose we have a barcode image taken in poor light conditions:<br />![bina1.PNG](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667279535081-a7dfa493-9047-4a0a-8ddc-cbcd94384c4a.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=134&id=u38788981&margin=%5Bobject%20Object%5D&name=bina1.PNG&originHeight=102&originWidth=226&originalType=binary&ratio=1&rotation=0&showTitle=true&size=30538&status=done&style=none&taskId=ub52ffe25-eb46-4b84-aa4f-6ab2d5aaa2a&title=unbalanced-lightening-image&width=296 "unbalanced-lightening-image")<br />If a global threshold(BM_THRESHOLD) is used to determine whether each pixel should be black or white, then the poorly lit white pixels in four corners might be treated as black pixels just like this:<br />![bina2.PNG](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667279535078-697ba151-c4c2-4999-8b4c-8497245b57ca.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=136&id=ud818cb49&margin=%5Bobject%20Object%5D&name=bina2.PNG&originHeight=103&originWidth=225&originalType=binary&ratio=1&rotation=0&showTitle=true&size=5644&status=done&style=none&taskId=u25e4c6ff-0d8e-4033-86ae-d045a11d4fd&title=use%20BM_THRESHOLD&width=296 "use BM_THRESHOLD")<br />Let's look at the BM_LOCAL_BLOCK. The principle is to split an image into a series of windows, each of which is in a processing unit (the size of the window is controlled by BlockSizeX and BlocksizeY, If BlockSizeX and BlockSizeY are not set manually, DBR will determine their default values based on the size of the image.), and then binarization is accomplished by a set of adaptive local thresholds in all these windows:<br />![bina3.PNG](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667279535090-808f7bb1-dd69-499e-a2b9-3007fc1d95b0.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=140&id=ud1b0dc25&margin=%5Bobject%20Object%5D&name=bina3.PNG&originHeight=106&originWidth=230&originalType=binary&ratio=1&rotation=0&showTitle=true&size=5643&status=done&style=none&taskId=u9c3a8003-7bd1-4297-b98b-2bde358d9a5&title=use%20BM_LOCAL_BLOCK%20with%20auto%20BLOCK%20size&width=304 "use BM_LOCAL_BLOCK with auto BLOCK size")<br />Setting appropriate BlockSizes result in a better-binarized image:<br />![bina4.PNG](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667279535102-4691ee1e-61a2-4851-a7ad-b37db3a85ca1.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=135&id=u82c0da7d&margin=%5Bobject%20Object%5D&name=bina4.PNG&originHeight=101&originWidth=228&originalType=binary&ratio=1&rotation=0&showTitle=true&size=5555&status=done&style=none&taskId=u24fc33fb-2826-477b-9196-717efc1e949&title=use%20BM_LOCAL_BLOCK%20with%208x8%20BLOCK%20size&width=305 "use BM_LOCAL_BLOCK with 8x8 BLOCK size")<br />Therefore, the images at the beginning of this section(with bad lighting conditions), BM_LOCAL_BLOCK is a better choice.<br />BM_THRESHOLD has the following arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| BinarizationThreshold | _int_ | [-1, 255] | -1 |
| ImagePreprocessingModesIndex | _int_ | [-1, 0x7fffffff] | -1 |

- For BM_THRESHOLD, we can explicitly set a value to the argument BinarizationThreshold which dictates at which point a pixel is regarded as black/white. Generally, we can just use the default value -1 which allows DBR to calculate a proper threshold itself.
- The index of a specific image preprocessing mode in the ImagePreprocessingModes parameter to which the current binarization mode is applied.

BM_LOCAL_BLOCK has the following arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| BlockSizeX | _int_ | [0, 1000] | 0 |
| BlockSizeY | _int_ | [0, 1000] | 0 |
| EnableFillBinaryVacancy | _int_ | [0, 1] | 1 |
| ImagePreprocessingModesIndex | _int_ | [-1, 0x7fffffff] | -1 |
| ThresholdCompensation | _int_ | [-255, 255] | 10 |

- For BM_LOCAL_BLOCK, the block size should be set to an appropriate value (5 ~ 8 times the module size) with the arguments BlockSizeX & BlockSizeY and EnableFillBinaryVacancy should be set to disable(0: disable; 1:enable). For example, the module size of QR Codes and Data Matrix Codes in this section are 29 pixels and 44 pixels respectively, this information can be derived from the IntermediateResults. If BlockSizeX & BlockSizeY are not set manually, DBR will determine their default values based on the size of the image.
- If the block size is not set properly, it's likely that vacancies will appear in the barcode modules in the binarized image in which case filling up the vacancies can help with the decoding. This operation is enabled with the argument EnableFillBinaryVacancy. However, filling up the vacancies is a time-consuming operation and should be avoided if speed is the top priority.
- ThresholdCompensation is a constant subtracted from the mean or weighted mean used for calculating the threshold. Normally, it is positive but may be zero or negative as well.

However BM_LOCAL_BLOCK algorithm is not always better than BM_THRESHOLD, you can refer to [another article](https://www.dynamsoft.com/barcode-reader/parameters/scenario-settings/how-to-set-binarization-modes.html?ver=latest#bm_local_block) for more information. 
<a name="hbzFH"></a>
##### 2.4 Choose the optimum localization modes
Now that we have a binarized image processed from the original image, we can start localizing the barcode zones. DBR comes with 8 options for the parameter LocalizationModes which determines how the localization is done. Of the 8 modes, 3 of them are designed for one or a few types of barcodes:

- LM_ONED_FAST_SCAN works best for linear or 1D barcodes that are of relatively high quality;
- LM_STATISTICS_MARKS is meant for QR or DataMatrix barcodes generated by direct part marking (DPM);
- LM_STATISTICS_POSTAL_CODE is meant for postal codes.

The following 2 modes are designed for certain usage scenarios:

- LM_SCAN_DIRECTLY can significantly speed things up in interactive scenarios (mostly for 1D, QR, or PDF417 codes);
- LM_STATISTICS_CENTRE is preferred when the barcodes appear at the center of the image (mostly for QR or DataMatrix codes).

The following 3 modes are designed for general use:

- LM_CONNECTED_BLOCKS usually gives the best results and should always be used as the top priority mode or at least as a backup mode;
- LM_LINES is a supplementary mode for LM_CONNECTED_BLOCKS and should always be used after LM_CONNECTED_BLOCKS;
- LM_STATISTICS is the last resort to try finding the barcode(s) when none or not enough barcodes have been found. Therefore, it should always be used as the last mode.

The last 3 modes can be used for all types of barcodes and they all share the same design: scan the image once to find all types of barcodes. And because DBR scans for the characteristics of all types of barcodes at once, it doesn't need to process the same image twice or more times when different types of barcodes are required. In other words, whether you just need to scan barcodes of a particular type or multiple types, the time cost is almost the same. For certain barcode type(s) or usage scenarios, choosing one or two localization modes can significantly speed things up, check out Adjust the localization modes on our recommended settings.

| Parameter | Default settings |
| --- | --- |
| [LocalizationModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/localization-modes.html#localizationmodes) | [LM_CONNECTED_BLOCKS, LM_SCAN_DIRECTLY, LM_STATISTICS, LM_LINES, 0, 0, 0, 0] |

In addition, when the target barcode shows more distinctive characteristics, a localization mode can be further customized for even better speed. Take LM_ONED_FAST_SCAN and LM_SCAN_DIRECTLY for examples???<br />LM_SCAN_DIRECTLY has the following arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | :---: | :---: | :---: |
| ScanStride | _int_ | [0, 0x7fffffff] | 0 |
| ScanDirection | _int_ | [0, 2] | 0 |
| IsOneDStacked | _int_ | [0, 1] | 0 |

- ScanStride sets the stride in pixels between scans when searching for barcodes. Default value 0 is to set ScanStride automatically by the library. When the set value is greater than half the width or height of the current image, the actual processing is 0. A bigger ScanStride means that the entire scan process is completed more quickly.
- You can also set a fixed ScanDirection:

0: Both vertical and horizontal directions.<br />1: Vertical direction.<br />2: Horizontal direction

- IsOneDStacked sets whether the oneD barcodes are stacked. False(0) by default.

LM_ONED_FAST_SCAN has the following arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | :---: | :---: | :---: |
| ScanStride | _int_ | [0, 0x7fffffff] | 0 |
| ScanDirection | _int_ | [0, 2] | 0 |
| ConfidenceThreshold | _int_ | [0, 100] | 60 |

- For ScanStride and ScanDirection, please refer to LM_SCAN_DIRECTLY mode.
- The localization results will be discarded if their confidence is less than the ConfidenceThreshold.

For all modes, if the default algorithm doesn't seem fast enough, you can change how it is done by specifying an alternative library with LibraryFileName and setting it up with LibraryParameters.
<a name="Ukfe6"></a>
##### 2.5 To further improve the barcode zones for decoding
After the localization, we have barcode zones located on an image. In this stage, these particular zones are cut from the image precisely on its boundaries. Then these images are preprocessed again before finally being passed (with barcode type information for each zone) to the next stage for decoding.<br />The preprocessing consists of two operations:
<a name="kGsm2"></a>
###### 2.5.1 Detect the color of the zones and adjust it based on BarcodeColourModes
| Parameter | Default settings |
| --- | --- |
| [BarcodeColourModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/barcode-colour-modes.html#barcodecolourmodes) | [BICM_DARK_ON_LIGHT,0,0,0,0,0,0,0] |

BarcodeColourModes currently supports two modes:

- BICM_DARK_ON_LIGHT
- BICM_DARK_ON_LIGHT_DARK_SURROUNDING

The usage as the name suggests, please refer to the actual scenario. These two modes have consistent arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| LightReflection | _int_ | [0, 1] | 1 |

- LightReflection determines if there is a light reflection on the barcode zone.

0: No light reflection.<br />1: Has light reflection.
<a name="HhGW5"></a>
###### 2.5.2 Detect the size of the zones and change it based on ScaleUpModes.
ScaleUpModes currently supports three modes:

- SUM_AUTO chooses a mode automatically by the library
- SUM_LINEAR_INTERPOLATION scales up using the linear interpolation method
- SUM_NEAREST_NEIGHBOUR_INTERPOLATION scales up using the nearest neighbor interpolation method
| Parameter | Default settings |
| --- | --- |
| [ScaleUpModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/scale-up-modes.html?ver=latest) | [SUM_AUTO,0,0,0,0,0,0,0] |

Suppose we have a 2x2 ultra-low-resolution image that needs to be scaled up to 4x4, then the gray areas need to be filled in with pixel values:<br />

<img src="https://cdn.nlark.com/yuque/0/2022/png/22760206/1667437147075-993044ab-2abe-40ae-ac04-238b0031674c.png" alt="image008" height="80%">
<br />

These two algorithms do the following respectively:

- SUM_LINEAR_INTERPOLATION: If the target pixel is on the line that connects two known pixels, then the value of the point is calculated by weighted distance and known pixel values.
- SUM_NEAREST_NEIGHBOUR_INTERPOLATION: Find the nearest known pixel for each target pixel and make a copy of its value.

The nearest neighbor interpolation algorithm is simpler and faster, but the image quality is often poor:

![3 - Copy.png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667290069556-77f20d6e-1d5c-494b-8160-47a4841c2f11.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=63&id=u953d6fcd&margin=%5Bobject%20Object%5D&name=3%20-%20Copy.png&originHeight=60&originWidth=110&originalType=binary&ratio=1&rotation=0&showTitle=true&size=2536&status=done&style=none&taskId=u57d026e2-976c-4b0b-bd1a-f2cf85a350a&title=Original%20image&width=115 "Original image")<br />
![scaleupnn.png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667290069543-aff1fb15-2842-46f5-a93e-666418dcb415.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=207&id=u88dd1fc2&margin=%5Bobject%20Object%5D&name=scaleupnn.png&originHeight=217&originWidth=375&originalType=binary&ratio=1&rotation=0&showTitle=true&size=13878&status=done&style=none&taskId=u8ee40721-e60b-44e4-8836-516cedf92fd&title=Scale%20up%20by%20nearest%20neighbor%20interpolation%20algorithm&width=357 "Scale up by nearest neighbor interpolation algorithm")![scaleupbn.png](https://cdn.nlark.com/yuque/0/2022/png/22760206/1667290069578-43eb01ec-6841-4187-b75d-035355c49188.png#clientId=u18aa4d7e-820d-4&crop=0&crop=0&crop=1&crop=1&from=ui&height=207&id=u45190113&margin=%5Bobject%20Object%5D&name=scaleupbn.png&originHeight=217&originWidth=375&originalType=binary&ratio=1&rotation=0&showTitle=true&size=17523&status=done&style=none&taskId=ud4502ca4-8edf-43ef-9067-a66e94e1c29&title=Scale%20up%20by%20linear%20interpolation%20algorithm&width=357 "Scale up by linear interpolation algorithm")<br />These two scale up modes have consistent arguments for further customizing:

| Argument name | Value Type | Value Range | Default Value |
| --- | --- | --- | --- |
| AcuteAngleWithXThreshold | _int_ | [-1, 90] | -1 |
| ModuleSizeThreshold | _int_ | [0, 0x7fffffff] | 0 |
| TargetModuleSize | _int_ | [0, 0x7fffffff] | 0 |

 The library automatically set the three arguments by default:

- AcuteAngleWithXThreshold sets the acute angle threshold for scale-up.
- ModuleSizeThreshold sets the module size threshold for scale-up.
- TargetModuleSize sets the target module size for scale-up.

If the module size of the barcode is smaller than the ModuleSizeThreshold and the acute angle with X of the barcode is larger than the AcuteAngleWithXThreshold, the barcode will be enlarged by a scale factor of N (the value of N is a power of 2) till N * module size >= TargetModuleSize.<br />Adjusting the color and the size of the barcode zone(s) can take some time but may help to improve the reading rate. If the target barcode image is not bad, consider skipping these processes(for interactive scenarios, it is recommended to skip).
<a name="r5uX5"></a>
##### 2.6 Expedite the actual barcode decoding
DBR can fix the partitioned barcodes (some are still deformed or incomplete). For best speed, we usually ignore these low-quality bar codes, that is do not set these parameters, and keep the default settings.

| Parameter | Default settings |
| --- | --- |
| [BarcodeComplementModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/barcode-complement-modes.html#barcodecomplementmodes) | [0,0,0,0,0,0,0,0] |
| [DeformationResistingModes](https://www.dynamsoft.com/barcode-reader/parameters/reference/deformation-resisting-modes.html#deformationresistingmodes) | [0,0,0,0,0,0,0,0] |

<a name="yqUAa"></a>
#### 3. Alternative methods
<a name="hRyYA"></a>
##### 3.1 Unleash the power of the CPU
The algorithm to process an image has quite a few steps and for each step, there could be multiple options to try. For processes that don't necessarily need to wait for each other, we can tell DBR to open multiple threads/workers to work on different tasks at the same time. <br />However, note that this is only meaningful on devices with a good CPU. On low-end desktops or mobile devices, it's better to limit the threads to 2 or even 1.

| Parameter | Default settings |
| --- | --- |
| [MaxAlgorithmThreadCount](https://www.dynamsoft.com/barcode-reader/parameters/reference/max-algorithm-thread-count.html) | 4 |

Other than the built-in multi-threading setting, another way to speed things up is to create multiple DBR instances and have them decode different images at the same time.
<a name="uCl2G"></a>
##### 3.2 Bypass time-consuming exceptions
Some challenging images may take a long time to decode, and more likely end in failure. So set a time(milliseconds) in Timeout to give up these exceptions. This process is particularly effective for video frames, because it makes no sense to spend too much time on one difficult frame when the next frame probably contains the same barcode(s).

| Parameter | Default settings |
| --- | --- |
| [Timeout](https://www.dynamsoft.com/barcode-reader/parameters/reference/time-out.html?ver=latest) | 10000 |

<a name="Z0wup"></a>
##### 3.3 Finish the reading prematurely
Barcode reading usually ends with the output of the content of the barcode. However, it's not always the intended behavior. This parameter can specify a certain stage to terminate the DBR algorithm. The main stages for the DBR algorithm are:

- TP_REGION_PREDETECTED
- TP_IMAGE_PREPROCESSED
- TP_IMAGE_BINARIZED
- TP_BARCODE_LOCALIZED
- TP_BARCODE_TYPE_DETERMINED
- TP_BARCODE_RECOGNIZED

Sometimes an application could be only interested in the coordinates of a barcode or even just know that a barcode exists on the image or frame. Therefore, we can tell DBR to finish the reading prematurely and return what has been found right away.

| Parameter | Default settings |
| --- | --- |
| [TerminatePhase](https://www.dynamsoft.com/barcode-reader/parameters/reference/terminate-phase.html) | TP_BARCODE_RECOGNIZED |

Here is a [demonstration of how to use TerminatePhase](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/scenario-settings/terminate.html?ver=latest).
<a name="gj8OA"></a>
##### 3.4 Avoid disk writing operations
Reading and writing disks is time-consuming, and differences between hard disks (HDD compared with SSD) can lead to even worse time-consuming performance. Try to avoid disk reads and writes.

| Parameter | Default settings |
| --- | --- |
| [IntermediateResultTypes](https://www.dynamsoft.com/barcode-reader/parameters/reference/intermediate-result-types.html) | IRT_NO_RESULT |
| [IntermediateResultSavingMode](https://www.dynamsoft.com/barcode-reader/parameters/reference/intermediate-result-saving-mode.html) | IRSM_MEMORY |

To control whether or not intermediate results should be collected, please set the IntermediateResultTypes to IRT_NO_RESULT (the default setting). If you choose to collect intermediate results, then the IntermediateResultSavingMode should be set to IRSM_MEMORY(save to memory)as it is less time-consuming than IRSM_FILESYSTEM(save to disk).
<a name="zP72E"></a>
##### 3.5 Fine-tune the performance further with FormatSpecification
The principle of acceleration is the same as 'Reduce the size of the image'(2.1). These are not set by default which means there is no limitation on the barcode shape. Set limitations on barcode searching for each type of barcode so that DBR can quickly skip uninterested zones. 

| Parameter | Default settings |
| --- | --- |
| [BarcodeAngleRangeArray](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/barcode-angle-range-array.html?ver=latest) | NULL |
| [BarcodeHeightRangeArray](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/barcode-height-range-array.html?ver=latest) | NULL |
| [BarcodeWidthRangeArray](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/barcode-width-range-array.html?ver=latest) | NULL |
| [BarcodeZoneBarCountRangeArray](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/barcode-zone-bar-count-range-array.html?ver=latest) | NULL |
| [ModuleSizeRangeArray](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/module-size-range-array.html?ver=latest) | 30 |

The parameters above can be set only via JSON template.

---

<a name="pLJTX"></a>
### Summary
In this article, we first talked about some general and effective ways to improve speed, then we went through the complete reading process and looked at most of the parameters that could impact speed. Lastly, we mentioned a few of the less traditional ways to speed things up.<br />Depending on the actual image you are scanning or the usage scenario you are trying to cope with, you can experiment with these parameters to find the most suitable settings for the best speed.

---

If you want to know the default value or the value range of RuntimeSettings, click [here](https://www.dynamsoft.com/barcode-reader/docs/server/programming/python/api-reference/class/PublicRuntimeSettings.html?ver=latest). See [Parameter Mode Enumeration](https://www.dynamsoft.com/barcode-reader/parameters/enum/parameter-mode-enums.html?ver=latest) for all mode members as well.<br />However, bear in mind that RuntimeSettings doesn't provide all the available configuration options of DBR. With a JSON template, you can make use of all the configuration options that DBR offers. See [all parameters](https://www.dynamsoft.com/barcode-reader/docs/core/parameters/reference/?ver=latest).<br />If you have any further questions, you can get in touch with [Dynamsoft Support](https://www.dynamsoft.com/company/contact/).<br />In reality, accuracy and read rate matter too. Read our other documents dedicated to these two topics:

- [How to boost Accuracy](https://www.dynamsoft.com/barcode-reader/performance/accuracy.html)
- [How to boost Read Rate](https://www.dynamsoft.com/barcode-reader/performance/read-rate.html)
