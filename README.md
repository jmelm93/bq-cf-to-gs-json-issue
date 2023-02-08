
# Issues w/ BigQuery Table Data >> Compressed Json via Cloud Function
## Overview of Files
    1. `exported_directly_from_bq_interface.json` - is json exported directly from BigQuery interface // is the correct format I'm looking to get to.
    2. `exported_via_cloud_function.json` - is what I'm getting (uncompressed) out of cloud function - which is just rows of objects? Not even a list? 
    3. `exported_via_cloud_function.json.gz` - is also what I get out of cloud function, which decompresses to what's seen in `exported_via_cloud_function.json`, which doesn't seem right to the best of my knowledge.
## Cloud Function Code: 
```
// https://cloud.google.com/bigquery/docs/samples/bigquery-extract-table-json#bigquery_extract_table_json-nodejs

const YYYYMMDD = new Date().toISOString().slice(0,10).replace(/-/g,"");
const bucketName = "XXXXXX.appspot.com";
const datasetId = "XXXXXX";
const tableId = "XXXXXXX";
const filenameCompressed = `/cache/${YYYYMMDD}/testing.json.gz`;
const filename= `/cache/${YYYYMMDD}/testing.json`;

const {BigQuery} = require('@google-cloud/bigquery');
const {Storage} = require('@google-cloud/storage');
const functions = require('@google-cloud/functions-framework');
// const {pako} = require('pako');

const bigquery = new BigQuery();
const storage = new Storage();

const options = {
    format: 'json',
    location: 'US',
};

const optionsGzip = {
        format: 'json',
        location: 'US',
        gzip: true,
};

functions.http('extractTableJSON', async (req, res) => {
    // Export data from the table into a Google Cloud Storage file
    const [job] = await bigquery
        .dataset(datasetId)
        .table(tableId)
        .extract(storage.bucket(bucketName).file(filename), options);

    // Export data to gzip
    const [jobCompressed] = await bigquery
        .dataset(datasetId)
        .table(tableId)
        .extract(storage.bucket(bucketName).file(filenameCompressed), optionsGzip);

    console.log(`jobCompressed ${jobCompressed.id} created.`);
    console.log(`Job ${job.id} created.`);
    res.send(`Job 1: ${job.id} created; Job 2: (Compressed) ${jobCompressed.id} created.`);

    // Check the job's status for errors
    const errors = job.status.errors;
    
    if (errors && errors.length > 0) {
        res.send(errors);
    }
});

```