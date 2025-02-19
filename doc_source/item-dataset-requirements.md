# Items dataset requirements \(custom\)<a name="item-dataset-requirements"></a>

 The data you provide for each item must match your Items dataset schema\. At minimum, you must provide an Item ID for each item \(max length 256 characters\)\. Depending on your schema, item metadata can include empty/null values\. Your schema must have minimum one metadata field, but if you add a `null` type, this value can be null for the item\. 

 To use categorical data, add a field of type `string` and set the field's categorical attribute to `true` in your schema\. Then include the categorical data in your bulk CSV file and incremental item imports\. For items with multiple categories, separate each value using the vertical bar, '\|'\. For example, for a GENRES field your data for an item might be action\|adventure\|comedy\. If you have a multiple levels of categorical data, add a field for each level and append a level indicator after each field name\. For example, CATEGORY\_L1, CATEGORY\_L2, CATEGORY\_L3\. 

Categorical values can have at most 1,000 characters\. If you have a user with a categorical value with more than 1,000 characters, your dataset import job will fail\. 

During model training, Amazon Personalize considers a maximum of 750,000 items\. If you import more than 750,000 items, Amazon Personalize decides which items to include in training, with an emphasis on including new items \(items you recently added with no interactions\) and existing items with recent interactions data\.

 For more information on minimum requirements and maximum data limits for an Items dataset, see [Service quotas](limits.md#limits-table)\.

## Items dataset schema example \(custom\)<a name="schema-examples-items"></a>

The following example shows how to structure an Items schema\. The `ITEM_ID` field is required\. The `GENRE` field is categorical metadata and the `DESCRIPTION` field is textual metadata\. At least one metadata field is required\. You can add a maximum of 50 metadata fields\. The `CREATION_TIMESTAMP` field is a reserved keyword\. For information about schema requirements, see [Custom dataset and schema requirements](custom-datasets-and-schemas.md#dataset-requirements)\.

```
{
  "type": "record",
  "name": "Items",
  "namespace": "com.amazonaws.personalize.schema",
  "fields": [
    {
      "name": "ITEM_ID",
      "type": "string"
    },
    {
      "name": "GENRES",
      "type": [
        "null",
        "string"
      ],
      "categorical": true
    },
    {
      "name": "CREATION_TIMESTAMP",
      "type": "long"
    },
    {
      "name": "DESCRIPTION",
      "type": [
        "null",
        "string"
      ],
      "textual": true
    },
  ],
  "version": "1.0"
}
```

For this schema, the first few lines of historical data in a CSV file might look like the following\.

```
ITEM_ID,GENRES,CREATION_TIMESTAMP,DESCRIPTION
1,Adventure|Animation|Children|Comedy|Fantasy,1570003267,"This is an animated movie that features action, comedy, and fantasy. Audience is children. This movie was released in 2004."
2,Adventure|Children|Fantasy,1571730101,"This is an adventure movie with elements of fantasy. Audience is children. This movie was release in 2010."
3,Comedy|Romance,1560515629,"This is a romantic comedy. The movie was released in 1999. Audience is young women."
4,Comedy|Drama|Romance,1581670067,"This movie includes elements of both comedy and drama as well as romance. This movie was released in 2020."
...
...
```