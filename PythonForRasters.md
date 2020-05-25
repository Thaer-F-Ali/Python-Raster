# Python-Raster
This program is for meaasuring health of vegetation in remote sensing using NDVI. This program measures NDVI for two images together.
from osgeo import gdal
import numpy as np

# Opening Raster
def openRaster(fn):
    ds= gdal.Open(fn)
    if ds is None:
        print("Error opening raster dataset")
    else:
        print("The file has been read")
    return ds
# Getting the Red and NIR bands of the raster
def getRasterBand(fn,band):
        ds = openRaster(fn)
        band = ds.GetRasterBand(band).ReadAsArray().astype(float)       
        return band

def createRasterFromCopy(ds, outfn, data, ndv= -9999.0):    
    geo_fn = ds.GetGeoTransform()
    proj_fn = ds.GetProjection()
    driver_tiff = gdal.GetDriverByName("GTiff")
    new_ds = driver_tiff.Create(outfn, xsize= ds.RasterXSize, ysize= ds.RasterYSize, bands= 1, eType= gdal.GDT_Float32)
    new_ds.SetGeoTransform(geo_fn)
    new_ds.SetProjection(proj_fn)
    new_ds.GetRasterBand(1).SetNoDataValue(ndv)
    new_ds.GetRasterBand(1).WriteArray(data)
    print("The file was created")
    new_ds = None
    ds = None
    
   
def ndvi(nirband, redband, ds):
    x = ds.RasterXSize
    y = ds.RasterYSize
    ndviband1 = np.full((y, x), -1.0)
    for i in range(0, y):
        for j in range(0, x):
            ndviband1[i][j] = (nirband[i][j]- redband[i][j])/ (nirband[i][j]+ redband[i][j])
            if ndviband1[i][j] < -1 or ndviband1[i][j] > 1:
                ndviband1[i][j] = -9999.0            
    return ndviband1
    
# Defining rasters names
infn1  = "G:/Veg_System/Wasit/Data/wasit20200213.tif"
infn2  = "G:/Veg_System/Wasit/Data/wasit20200417.tif"
outfn1 = "G:/Veg_System/Wasit/Data/NDVI-20200213.tif"
outfn2 = "G:/Veg_System/Wasit/Data/NDVI-20200417.tif"
ds1 = openRaster(infn1)
ds2 = openRaster(infn2)
    
# NDVI for the first raster
redband1 = getRasterBand(infn1, 4)
nirband1 = getRasterBand(infn1, 5)
ndviband1 = ndvi(nirband1, redband1, ds1)
createRasterFromCopy(ds1, outfn1, ndviband1)

# NDVI for the second raster
redband2 = getRasterBand(infn2, 4)
nirband2 = getRasterBand(infn2, 5)
ndviband2 = ndvi(nirband2, redband2, ds2)
createRasterFromCopy(ds2, outfn2, ndviband2)

print("End of the Program")
