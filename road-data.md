### get road data
- BBBike
- - https://extract.bbbike.org
- shp
- send email




- Overpass Turbo
- https://overpass-turbo.eu/
- select box
- code/Wizard
- node
  highway=*
  ({{bbox}});
out;
-Export /GeoJSON

### QGIS  转为 DXF / DWG
- Layer → Add Layer → Add Vector Layer
- roads.shp（道路）
- roads（裁剪后的小范围） → Export → Save Features As…
- Layer → Add Layer → Add Vector Layer 打开 GeoJSON
