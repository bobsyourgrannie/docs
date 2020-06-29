



> Persist report rendering outputs for later access

## Basics
Everytime you call jsreport API to render report, you will get back stream with report content. This is usefull when you want to display report directly to the user or you want to store the result in your data storage for later use. But sometimes you just don't want the report in the time of generation and you don't want to store it on your own. In this case you can use `Reports` extension and let the jsreport store the report for you.

The reports extension uses [blob storage](/learn/blob-storages) abstraction for reports blobs persistence. This by default stores reports to the file system. Check the documentation if you want to store reports to a different destination.

## API

Reports extension is not used by default for all rendering requests and you need to specify it for every request separately.

To use `reports` extension you need extend rendering request in a following way:

> `POST:` https://jsreport-host/api/report
> `BODY:`
>```js
   {
      "template": { "shortid" : "g1PyBkARK" },
      "data" : { ... },
      "options": {
	      "reports": { "save": true }
      }
   }
>```

This will create a `Report` entity you can use in jsreport studio as well as in OData API. Additionally it will add custom header `Permanent-Link` to the response which you can later use to actually download the report content.

The stored report entity name is by default inherited from the template name. However, you can customize it by passing`{ "options": { "reportName": "yourCustomName" }` in the request body.

The name of the stored file/blob is by default inherited from the entity unique _id. This can be customized using `{ "options": { "reports": { "save": true, "blobName": "myfilename" } }`

## Async
Sending a rendering request with `options.reports.save = true` will instruct the extension to save the report and add `Permanent-Link` header to the response, but the rendering is still synchronous and you receive the response back after the process is finished. If you want to start the rendering process asynchronously and receive the response immediately you should set `options.reports.async = true`.

> `POST:` https://jsreport-host/api/report
> `BODY:`
>```js
   {
      "template": { "shortid" : "g1PyBkARK" },
      "data" : { ... },
      "options": {
	      "reports": { "async": true }
      }
   }
>```

In this case you receive response with `Location` header containing the url to the rendering status page. It will appear in a form like `http://jsreport-host/reports/id/status`. You can then ping the status page to check if the rendering is done. In that case the response status will be `201` and the location header will contain the address to the stored report.

## Cleanup
The reports stored from async calls persist forever by default. You can change this and enable automatic clean up of old reports. This can be done through config.
```json
{ 
  "extensions": { 
    "reports": {
      // how often the cleanup runs
      "cleanInterval": "5m",
      // how much old reports should be deleted
      "cleanTreshold": "1d"
    }
  }
}
```
Be aware that the auto-cleanup logic also removes the reports produced through scheduling extension.

## OData

You can use standard OData API to manage and query report entities. For example you can query all reports using:
> `GET` http://jsreport-host/odata/reports
