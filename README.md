  <h1>JupyterユーザーのためのKepler.glガイド</h1>

  - [Install](#install)
  - [1. kepler.gl Mapのロード](#1-load-keplergl-map)
    - [`KeplerGl()`](#keplergl)
  - [2. Add Data](#2-add-data)
    - [`.add_data()`](#add_data)
    - [`.data`](#data)
  - [3. Data Format](#3-data-format)
    - [`CSV`](#csv)
    - [`GeoJSON`](#geojson)
    - [`DataFrame`](#dataframe)
    - [`GeoDataFrame`](#geodataframe)
    - [`WKT`](#wkt)
  - [4. mapのカスタマイズ](#4-customize-the-map)
  - [5. Save and load config](#5-save-and-load-config)
    - [`.config`](#config)
  - [6. Match config with data](#6-match-config-with-data)
  - [7. Save Map](#7-save-map)
    - [`.save_to_html()`](#save_to_html)
  - [例](#demo-notebooks)

<br></br>
## Install
### Prerequisites
- Python >= 2
- ipywidgets >= 7.0.0

pipを用いてinstall:

    $ pip install keplergl


Macかつ`pip install`を用いており、Notebook >= 5.3の場合は以下のコードを入力する必要はない

    $ jupyter nbextension install --py --sys-prefix keplergl # can be skipped for notebook 5.3 and above

    $ jupyter nbextension enable --py --sys-prefix keplergl # can be skipped for notebook 5.3 and above

#### Jupyter Labを使用している場合
以下のJupyetr extensionと[node](https://nodejs.org/en/download/package-manager/#macos) `> 8.15.0`が必要となる

[Homebrew](https://brew.sh/) on Macを使用している場合:

    $ brew install node@8

install jupyter labextension.

    $ jupyter labextension install @jupyter-widgets/jupyterlab-manager keplergl-jupyter

**Prerequisites for JupyterLab:**
- Node > 8.15.0
- Python 3
- JupyterLab>=1.0.0

## 1. keplergl mapのロード
### `KeplerGl()`

- Input:
    - __`height`__  _optional_ default: `400`

        表示されるマップの高さ

    - __`data`__ `dict` _optional_

        Datasets as a dictionary, key is the name of the dataset. Read more on [Accepted data format][data_format]

    - __`config`__ `dict` _optional_
        Map config as a dictionary. The `dataId` in the layer and filter settings should match the `name` of the dataset they are created under

セルに kepler.gl widgetを読み込んでみる.



```python
# Load an empty map
from keplergl import KeplerGl
map_1 = KeplerGl()
map_1
```

![empty map][empty_map]

dataとconfigを通したい場合は[match config with data](match-config-w-data)を参照

```python
# Load a map with data and config and height
from keplergl import KeplerGl
map_2 = KeplerGl(height=400, data={"data_1": my_df}, config=config)
map_2
```

![Load map with data and config][load_map_w_data]

<br></br>

## 2. Add Data
### `.add_data()`
- Inputs
    - __`data`__ _required_ CSV, GeoJSON or DataFrame. Read more on [Accepted data format][data_format]
    - __`name`__ _required_  data entryの名前.

`name` of the datasetは `dataId` property of each `layer`,  `filter` and `interactionConfig` in the configに用いれる.

```python
# DataFrame
df = pd.read_csv('hex-data.csv')
map_1.add_data(data=df, name='data_1')

# CSV
with open('csv-data.csv', 'r') as f:
    csvData = f.read()
map_1.add_data(data=csvData, name='data_2')

# GeoJSON as string
with open('sf_zip_geo.json', 'r') as f:
    geojson = f.read()

map_1.add_data(data=geojson, name='geojson')
```

![Add data to map][map_add_data]

### `.data`
`Dict`としてdataを出力

```python
map_1.data
# {'data_1': 'hex_id,value\n89283082c2fffff,64\n8928308288fffff,73\n89283082c07ffff,65\n89283082817ffff,74\n89283082c3bffff,66\n8...`,
#  'data_3': 'location, lat, lng, name\n..',
#  'data_3': '{"type": "FeatureCollecti...'}
```

<br></br>

## 3. Data Format
kepler.gl supports **CSV**,  **GeoJSON**, Pandas **DataFrame** or GeoPandas **GeoDataFrame**.

### `CSV`
```python
with open('csv-data.csv', 'r') as f:
    csvData = f.read()
# csvData = "hex_id,value\n89283082c2fffff,64\n8928308288fffff,73\n89283082c07ffff,65\n89283082817ffff,74\n89283082c3bffff,66\n8..."
map_1.add_data(data=csvData, name='data_2')
```

### `GeoJSON`
 [GeoJSON Specification (RFC 7946)][geojson]より: 
 - GeoJSONはgeographic data structuresをエンコーディングするためのformat。
 - GeoJSON object とは a region of space (a `Geometry`), a spatially bounded entity (a Feature), a list of Features (a `FeatureCollection`)などを表す 
 - GeoJSONがサポートするgeometry typeは以下:
   - `Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`, and `GeometryCollection`. 
- `GeometryCollection`以外のsingle [`Feature`][features]、[`FeatureCollection`][feature_collection]をkepler.glはサポートしてる. `GeoJSON` を `string` または `dict` typeとしてフォーマットすることもできる



```python
feature = {
    "type": "Feature",
    "properties": {"name": "Coors Field"},
    "geometry": {"type": "Point", "coordinates": [-104.99404, 39.75621]}
}

featureCollection = {
    "type": "FeatureCollection",
    "features": [{
        "type": "Feature",
        "geometry": {"type": "Point", "coordinates": [102.0, 0.5]},
        "properties": {"prop0": "value0"}
    }]
}

map_1.add_data(data=feature, name="feature")
map_1.add_data(data=featureCollection, name="feature_collection")
```

```python
# GeoJson Feature geometry
geometryString = {
    'type': 'Polygon',
    'coordinates': [[[-74.158491,40.835947],[-74.148473,40.834522],[-74.142598,40.833128],[-74.151923,40.832074],[-74.158491,40.835947]]]
}

# create json string
json_str = json.dumps(geometryString)

# create data frame
df_with_geometry = pd.DataFrame({
    'id': [1],
    'geometry_string': [json_str]
})

# add to map
map_1.add_data(df_with_geometry, "df_with_geometry") # dataとname
```

### `DataFrame`
[pandas.DataFrame][data_frame]も扱うことができる
```python
df = pd.DataFrame(
    {'City': ['Buenos Aires', 'Brasilia', 'Santiago', 'Bogota', 'Caracas'],
     'Latitude': [-34.58, -15.78, -33.45, 4.60, 10.48],
     'Longitude': [-58.66, -47.91, -70.66, -74.08, -66.86]})

w1.add_data(data=df, name='cities')
```

### `GeoDataFrame`
 [geopandas.GeoDataFrame][geo_data_frame]とは自動的に `geometry` column をshapelyからwkt stringに変換してくれる.
```python
url = 'http://eric.clst.org/assets/wiki/uploads/Stuff/gz_2010_us_040_00_500k.json'
country_gdf = geopandas.read_file(url)
w1.add_data(data=country_gdf, name="state")
```

![US state][geodataframe_map]

### `WKT`

```python
# WKT
wkt_str = 'POLYGON ((-74.158491 40.835947, -74.130031 40.819962, -74.148818 40.830916, -74.151923 40.832074, -74.158491 40.835947))'

df_w_wkt = pd.DataFrame({
    'id': [1],
    'wkt_string': [wkt_str]
})

map_1.add_data(df_w_wkt, "df_w_wkt")
```


<br></br>

## 4. Customize the map

mapを表示したまま保存したい場合はカーネルシャットダウンする前に `Widgets > Save Notebook Widget State`

![Map interaction][map_interaction]

<br></br>

## 5. Save and load config

### `.config`
map configurationの出力は以下；
```python
map_1.config
## {u'config': {u'mapState': {u'bearing': 2.6192893401015205,
#  u'dragRotate': True,
#   u'isSplit': False,
#   u'latitude': 37.76209132041332,
#   u'longitude': -122.42590232651203,
```

mapが完成したらその時にconfigを出力する。参照：[match config with data](match-config-w-data).

#### configの活用例

 1. configをthe mapに適用.
```python
config = {
    'version': 'v1',
    'config': {
        'mapState': {
            'latitude': 37.76209132041332,
            'longitude': -122.42590232651203,
            'zoom': 12.32053899007826
        }
        ...
    }
},
map_1.add_data(data=df, name='data_1')
map_1.config = config
```
 2. map作成時にconfig適用
```python
map_1 = KeplerGl(height=400, data={'data_1': my_df}, config=config)
```
- ファイルにconfigを書き込み`%run`コマンド実行時に再現されるようにする
```python
# Save map_1 config to a file
with open('hex_config.py', 'w') as f:
   f.write('config = {}'.format(map_1.config))

# load the config
%run hex_config.py
```

<br></br>
## 6. Match config with data

  - `dataId` of `layer.config`,
  - `dataId` of `filter`
  - key in `interactionConfig.tooltip.fieldToShow`.

![Connect data and config][connect_data_config]

<br></br>
## 7. Save Map

上述と同様に`Widget > Save Notebook Widget State` .

![Save Widget State][save_widget_state]

### `.save_to_html()`

- input
    - **`data`**: _optional_  A data dictionary {"name": data}, if not provided, will use current map data
    - **`config`**: _optional_ map config dictionary, if not provided, will use current map config
    - **`file_name`**: _optional_ the html file name, default is `keplergl_map.html`
    - **`read_only`**: _optional_ if `read_only` is `True`, hide side panel to disable map customization

You can export your current map as an interactive html file.

```python
# this will save current map
map_1.save_to_html(file_name='first_map.html')

# this will save map with provided data and config
map_1.save_to_html(data={'data_1': df}, config=config, file_name='first_map.html')

# this will save map with the interaction panel disabled
map_1.save_to_html(file_name='first_map.html' read_only=True)
```
# Demo Notebooks
- [Load kepler.gl](https://github.com/keplergl/kepler.gl/blob/master/bindings/kepler.gl-jupyter/notebooks/Load%20kepler.gl.ipynb): Load kepler.gl widget, add data and config
- [Geometry as String](https://github.com/keplergl/kepler.gl/blob/master/bindings/kepler.gl-jupyter/notebooks/Geometry%20as%20String.ipynb): Embed Polygon geometries as `GeoJson` and `WKT` inside a `CSV`
- [GeoJSON](https://github.com/keplergl/kepler.gl/blob/master/bindings/kepler.gl-jupyter/notebooks/GeoJSON.ipynb): Load GeoJSON to kepler.gl
- [DataFrame](https://github.com/keplergl/kepler.gl/blob/master/bindings/kepler.gl-jupyter/notebooks/DataFrame.ipynb): Load DataFrame to kepler.gl
- [GeoDataFrame](https://github.com/keplergl/kepler.gl/blob/master/bindings/kepler.gl-jupyter/notebooks/GeoDataFrame.ipynb): Load GeoDataFrame to kepler.gl

[jupyter_widget]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_widget.png
[empty_map]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_empty_map.png
[geodataframe_map]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_geodataframe.png
[map_interaction]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_custom_map.gif
[load_map_w_data]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_widget.png
[map_add_data]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_add_data.png
[connect_data_config]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_connect_data_w_config.png
[save_widget_state]: https://d1a3f4spazzrp4.cloudfront.net/kepler.gl/documentation/jupyter_save_state.png

[wkt]: https://dev.mysql.com/doc/refman/5.7/en/gis-data-formats.html#gis-wkt-format
[geojson]: https://tools.ietf.org/html/rfc7946
[feature_collection]: https://tools.ietf.org/html/rfc7946#section-3.3
[features]: https://tools.ietf.org/html/rfc7946#section-3.2
[data_frame]: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html
[geo_data_frame]: https://geopandas.readthedocs.io/en/latest/data_structures.html#geodataframe

[match-config-w-data]: #match-config-with-data
[data_format]: #3-data-formatv
