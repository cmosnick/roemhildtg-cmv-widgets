# Relationship Table Tabs

A tabbed widget for displaying and interacting with tables related to feature layers. Includes a [child widget RelationshipTable/RelationshipTable](#relationship-table-widget) to use in other situations, like the identify popup (see below).

Intended for use in [CMV v1.3.3](https://github.com/cmv/cmv-app/)

## Requirements

- Layer mode should be set to `ON_DEMAND` or `SNAPSHOT`. `SELECTION` will not work currently.
- Ensure [a proxy](https://github.com/Esri/resource-proxy) is configured and working

## Config

### CMV pane and widget config

```javascript
panes: {
    left: {
        splitter: true
    }
    bottom: {
        id: 'sidebarBottom',
        placeAt: 'outer',
        splitter: true,
        collapsible: true,
        region: 'bottom',
        style: 'height:300px;display:none;',
        content: '<div id="relatedRecords" style="height:100%;"></div>'
    }
}
```

```javascript
widgets: {
//....
relatedRecords: {
    include: true,
    id: 'relatedRecords',
    type: 'domNode',
    srcNodeRef: 'relatedRecords',
    path: 'gis/CMV_Widgets/widgets/RelationshipTableTabs',
    title: 'Related Records',
    options: {
        //required option
        layerControlLayerInfos: true,

        //optional relationships property
        relationships: {
            <layerID)>: { //layerID (string) key refers to featurelayer id in the operationalLayers array
                <relationshipID>: { //relationshipID (integer) key referrs to the relationship id on the rest services page
                    //relationship tab title
                    title: 'Inspections',

                    //set exclude to true to skip this relationship
                    exclude: false,

                    //other dgrid options like columns may be included
                }
            }
        }
    }
}
//.....
}
```

## Widget Options:

Key              | Type       | Default | Description
---------------- | ---------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`layerInfos`     | `boolean`  | `null`  | set layerControlLayerInfos to true, a layerInfos object is required
`relationships`  | `{object}` | `{}`    | An object describing each relationship. Details below
`tabPosition`    | `string`   | `'top'` | dijit tabContainer.tabPosition property: `'top'`, `'left-h'`, Not working due to layout issues: `'bottom'`, `'right-h'`
`tabContainerId` | `string`   | `null`  | An optional id of a widget that is a tabContainer or contains a tabContainer property. If specified, this widgets tables will be added to an existing tabContainer like the one developed by tmcgee [here](https://github.com/tmcgee/cmv-widgets/blob/master/widgets/AttributesTable/README.md) instead of creating a new one.

### `relationships` property

Each relationship object may have the following properties as well as [properties used by dgrid](https://github.com/SitePen/dgrid/blob/master/doc/components/core-components/OnDemandList-and-OnDemandGrid.md)

Key       | Type      | Default                        | Description
--------- | --------- | ------------------------------ | ---------------------------------------------------------------------------
`title`   | `string`  | layer.name - relationship.name | the title for the relationship tab
`exclude` | `boolean` | `null`                         | by default all relationships are included. set exclude: false to avoid this

You may also use this widget in a programatic matter:

```javascript
new RelationshipTableTabs({
  layerInfos: [{
      id: 'demographics',
      layer: demographicsLayer
  }]
  //other options
}, 'domNodeId');
```

# Relationship Table Widget

This class can be used standalone in situations where you don't want tabbed tables, like the identify widget.

```javascript
new RelationshipTable({
  attributes: data.attributes,
  style: 'width:100%;',
  title: 'Bridge Links',
  objectIdField: 'OBJECTID',
  relationshipId: 0,
  url: '/arcgis/rest/services/Apps/RelatedRecords/MapServer/0',
  columns: [{
    label: 'Link',
    field: 'Link_URL',
    formatter: formatters.url
  }, {
    label: 'Category',
    field: 'Category'
  }]
 }, 'domNodeId');
```

## Constructor Options

In addition to the options below, RelationshipTable inherits from `_WidgetBase`, `OnDemandGrid`, `ColumnHider` and `DijitRegistry`.

[See dgrid docs for additional useful properties](https://github.com/SitePen/dgrid/blob/v1.1.0/doc/components/core-components/OnDemandList-and-OnDemandGrid.md)

Key                    | Type       | Default                                                     | Description
---------------------- | ---------- | ----------------------------------------------------------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
`objectIdField`        | `String`   | `null`                                                      | The object id field for this feature layer. This is typically the `layer.objectIdField` property. If you have an alias on the object id field, use the alias.
`attributes`           | `{object}` | `null`                                                      | A map of attributes returned from an identify result. This is typically the feature.attributes property. It must have an property with the name of the `objectIdField`.
`url`                  | `String`   | `''`                                                        | The url for the feature layer.
`relationshipId`       | `Number`   | `0`                                                         | The id of the relationship to use. This is found on the rest services page for the feature layer, by adding `?f=json` to the url.
`defaultNoDataMessage` | `String`   | `'No results.'`                                             | The default message to display when there is no data.
`errorDataMessage`     | `String`   | `'The query could not be executed. Is a proxy configured?'` | The default error message to display.
`loadingMessage`       | `String`   | `'Loading...'`                                              | The default error message to display.

## Include in identify popup:

Related Records comes with a simple content formatter for cmv. Add the required file to your `identify.js` and use it to include relationship queries in your popup.

The factory is a function where you pass an array of objects. Each object needs the properties defined in the constructor options above.

```javascript
define([
  'roemhildtg/RelationshipTable/identify/factory'
], function (factory) {


//.........other identify.js stuff
identifies: {
  layerId: { // operational layers layer `id`
    0: {
        title: 'Bridge Inventory',
        content: factory([{
            title: 'Bridge Links',
            objectIdField: 'OBJECTID',
            relationshipId: 0,
            url: '/arcgis/rest/services/Apps/RelatedRecords/MapServer/0',
            columns: [{
                    label: 'Link',
                    field: 'Link_URL'
                }, {
                    label: 'Category',
                    field: 'Category'
                }]
        }// , { ... more relationships } if your dynamic layer has more relationships, you can add more than one
        ])
    }
  }
}
```

## Changes:

3/6/2017: Add an identify content factory

5/30/2015: Major widget changes

- moved majority of relationship querying and display code to a self-contained class RelationshipTable
- renamed class to RelationshipTableTabs to better describe the widget
- renamed widget folder to RelationshipTable
- renamed columnInfos property to relationships

**RelationshipTable Class**

- Can be used standalone (like in identify popup) or with the RelatedRecordTabs
- Inherits all properties, methods and events from `OnDemandGrid`, `ColumnHider`, and `DijitRegistry`
