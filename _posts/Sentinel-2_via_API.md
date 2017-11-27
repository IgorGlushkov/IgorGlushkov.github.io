
### Get Sentinel data via API


```python
from sentinelsat.sentinel import SentinelAPI
from sentinelsat import read_geojson, geojson_to_wkt
import json,os,glob
import zipfile
import shutil
```

#### Authentification


```python
#set api 
s2_api = SentinelAPI(
    user="igglu",
    password="vorron78",
    api_url="https://scihub.copernicus.eu/apihub"
)
```

#### GET AOI


```python
area = geojson_to_wkt(read_geojson('Box1.geojson'))
```

#### SET QUERY


```python
#query
try:
    products = s2_api.query(
        area,
        date = ('20170801', '20171210'),
        platformname = "Sentinel-2",
        cloudcoverpercentage=(0,50)
        )
except:
    print("No connection")
```

    No connection
    

#### Get imageries coverage


```python
#convert to geojson
try:
    gj = s2_api.to_geojson(products)
    #save
    with open('footprint3.geojson', 'w') as f:
        json.dump(gj, f)
except:
    print("no data found")
```

    no data found
    

#### Get result as data frame


```python
#convert to 
products_df = s2_api.to_dataframe(products)

#sort and limit to first 5 sorted products
products_df_sorted = products_df.sort_values(['cloudcoverpercentage', 'ingestiondate'], ascending=[True, True])
products_df_sorted = products_df_sorted.head(3)
print(products_df_sorted)
```

#### Download first 3


```python
try:
    s2_api.download_all(products_df_sorted['uuid'])
except:
    print("Error downloading")
```

    Error downloading
    

#### Unzip to separate folder


```python
def unzip():
    scenes = glob.glob('*.zip')
    for scene in scenes:
        try:
            a = zipfile.ZipFile(scene)
            scene_name = scene.split('_')[0]+'_'+scene.split('_')[2]+'_'+scene.split('_')[3]+'_'+scene.split('_')[4]
            a.extractall(path=scene_name)
            a.close()
        except IOError:
            False
unzip()
```

#### Create TIF


```python
#buildtif
bandlist=("B12.jp2","B08.jp2","B04.jp2")
datadir="d:/QGIS/materials/IND_Day4/Session15/jupyter"
def buildvrt():
    for subdir, dirs, files in os.walk(datadir):
        bands=glob.glob(os.path.join(subdir, '*B*.jp2'))
        scenes=[]
        if bands == []:
            print("no bands found")
            continue
        else:
            #print(bands)
            count=0
            scenes.append(("_").join(os.path.basename(bands[0]).split('.')[0].split('_')[0:2]))
            list=[]
            for band in bands:
                if str(os.path.basename(band).split('_')[2]) in bandlist:
                        list.append(band)
                else:
                    print("No match found")
                    continue
            output=os.path.join(datadir,str(scenes[count])+'.tif')
            if list:
                command= 'gdal_merge -separate -co COMPRESS=lzw -ot Int16 -o %s %s' % (output,(' ').join(list))
                print command
                os.system(command)
                command= 'gdaladdo -ro %s 2 4 8 16 32 64 128 256 512 ' % (output)
                print command
                os.system(command)
            else:
                print("list is empty")
                continue            
            count+=1
buildvrt()
```

    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    gdal_merge -separate -co "COMPRESS=lzw" -ot Int16 -o d:/QGIS/materials/IND_Day4/Session15/jupyter\T48MVU_20171005T030551.tif d:/QGIS/materials/IND_Day4/Session15/jupyter\S2A_20171005T030551_N0205_R075\S2A_MSIL1C_20171005T030551_N0205_R075_T48MVU_20171005T032534.SAFE\GRANULE\L1C_T48MVU_A011941_20171005T032534\IMG_DATA\T48MVU_20171005T030551_B04.jp2 d:/QGIS/materials/IND_Day4/Session15/jupyter\S2A_20171005T030551_N0205_R075\S2A_MSIL1C_20171005T030551_N0205_R075_T48MVU_20171005T032534.SAFE\GRANULE\L1C_T48MVU_A011941_20171005T032534\IMG_DATA\T48MVU_20171005T030551_B08.jp2 d:/QGIS/materials/IND_Day4/Session15/jupyter\S2A_20171005T030551_N0205_R075\S2A_MSIL1C_20171005T030551_N0205_R075_T48MVU_20171005T032534.SAFE\GRANULE\L1C_T48MVU_A011941_20171005T032534\IMG_DATA\T48MVU_20171005T030551_B12.jp2
    gdaladdo -ro d:/QGIS/materials/IND_Day4/Session15/jupyter\T48MVU_20171005T030551.tif 2 4 8 16 32 64 128 256 512 
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    no bands found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    No match found
    gdal_merge -separate -co "COMPRESS=lzw" -ot Int16 -o d:/QGIS/materials/IND_Day4/Session15/jupyter\T48MVB_20171025T030811.tif d:/QGIS/materials/IND_Day4/Session15/jupyter\S2A_20171025T030811_N0206_R075\S2A_MSIL1C_20171025T030811_N0206_R075_T48MVB_20171025T100221.SAFE\GRANULE\L1C_T48MVB_A012227_20171025T032827\IMG_DATA\T48MVB_20171025T030811_B04.jp2 d:/QGIS/materials/IND_Day4/Session15/jupyter\S2A_20171025T030811_N0206_R075\S2A_MSIL1C_20171025T030811_N0206_R075_T48MVB_20171025T100221.SAFE\GRANULE\L1C_T48MVB_A012227_20171025T032827\IMG_DATA\T48MVB_20171025T030811_B08.jp2 d:/QGIS/materials/IND_Day4/Session15/jupyter\S2A_20171025T030811_N0206_R075\S2A_MSIL1C_20171025T030811_N0206_R075_T48MVB_20171025T100221.SAFE\GRANULE\L1C_T48MVB_A012227_20171025T032827\IMG_DATA\T48MVB_20171025T030811_B12.jp2
    gdaladdo -ro d:/QGIS/materials/IND_Day4/Session15/jupyter\T48MVB_20171025T030811.tif 2 4 8 16 32 64 128 256 512 
    No match found
    list is empty
    no bands found
    no bands found
    

#### Clip with AOI


```python
aoi="Box1.geojson"
def clipvrt():
    scenes=glob.glob(os.path.join(datadir, '*.tif'))
    if scenes == []:
        print('no files')
    else:
        for scene in scenes:
            output=os.path.join(datadir, str(os.path.basename(scene).split('.')[0]+'clip.tif'))   
            command= 'gdalwarp -q -cutline %s -crop_to_cutline  -co COMPRESS=lzw -ot Int16 -of GTiff %s %s' % (aoi,scene,output)
            os.system(command)           
#clipvrt()
```

#### Remove temp directories


```python
def rm():
    flist=os.listdir('.')
    for root,dirs,files in os.walk(datadir):
        for d in dirs:
            if "." not in d:
                shutil.rmtree(os.path.join(root,d),ignore_errors=True)
            else:
                print("is not a directory")
            
rm()
```

    is not a directory
    
