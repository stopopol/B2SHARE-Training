# B2SHARE REST API Hands-on Exercises
<img width="25%" height="25%" align="right" src="img/EUDATSummerSchool.png" alt="EUDAT Summer School logo" text="EUDAT Summer School logo">
This document contains several exercises that introduce the core functionality of the B2SHARE REST API.

### Prerequisites
To make the exercises the following is needed:
- Internet connection
- Python 2.7+ installed
- Packages installed:
 - requests
 - simplejson
 - jsonpatch

Packages can be installed using the PIP tool, see [here](https://pip.pypa.io/en/stable/installing/) on how to install this tool.

### Requests
Requirements for each request:
- Request URL and HTTP method (e.g. GET, PUT)
- Object identifiers when loading object states (e.g. record, community)

Optional:
- Additional parameters (e.g. your token)
- Data payloads (e.g. files or text)

After each request, check the status of your request by printing the request object or its status code specifically:

```python
>>> r = requests.get('https://trng-b2share.eudat.eu/api/communities')
>>> print r
<Response [200]>
>>> print r.status_code
200
```

If the status code is in the range of 300 to 599, the request failed and needs to be investigated:

```python
>>> print r
<Response [404]>
>>> print r.reason
NOT FOUND
```

### Handling response data
In general, every request made to the B2SHARE service will return a HTTP response code and a text, independent of whether the request was successful or completely understood.

The response text is always in the JSON format and can be interpreted using the `json` package, e.g.:

```python
>>> r = requests.get('https://trng-b2share.eudat.eu/api/communities')
>>> res = json.loads(r.text)
```

or

```python
>>> res = r.json()
>>> print res['hits']['total']
12
```

### Access token loading
For some of the requests, a personal access token is required. This token can be generated on the [B2SHARE website](https://trng-b2share.eudat.eu) by logging in and navigating to your profile page.

Copy the token to the clipboard and store it in a file:

```sh
$ echo "token here" > token.txt
```

Then load it in your Python session:

```python
>>> f = open(‘token.txt’, ‘r’)
>>> token = f.read().strip()
>>> print token
<token>
```

### Data retrieval
B2SHARE stores data in separate objects of which each object represents a record, community or other entity. Each object has its own identifier which uniquely identifies the object and makes it possible to refer to it from other objects and using it in URLs. So to retrieve the data of a specific object, this identifier needs to be added to the URL of the request.

## Exercises
The exercises in this section can be done using Python, cURL or directly in the browser. Only the creation of new records cannot easily be done in the browser directly through the API, use Python or cURL for this.

### Records
In this exercise the information and metadata of a single record will be retrieved and processed. In addition, the file contained in the record is downloaded and checked.

#### Exercise 1a Retrieve single record info
Retrieve the metadata and other information of a single record. You can choose any record on the [training website](https://trng-b2share.eudat.eu), or use the record ID shown below.

- Endpoint: `/api/records/<RECORD_ID>`
- Method: GET
- Response: 200

Data:
- `RECORD_ID`: 47077e3c4b9f4852a40709e338ad4620

Tasks:
- Create the URL that retrieves the record data and store in the variable `url`

```python
>>> url = 'https://trng-b2share.eudat.eu/api/records/47077e3c4b9f4852a40709e338ad4620'
```

- Use the `requests` package to retrieve the object data and optionally the `simplejson` package to format the output of the response text:

```python
>>> import requests, simplejson
```

- Execute the request, check whether the request succeeded and store the response text in the variable `res`

#### Exercise 1b Check metadata and included files
Investigate the metadata and files contained in the record using the results obtained in the previous exercise.

Use the results of the previous exercise.

Tasks:
- Find the ePIC PID of the record:

```python
>>> print res['metadata']['ePIC_PID']
http://hdl.handle.net/11304/619eda56-100f-43f0-9e72-98a22792eb25
```

- Display the value of the title, author and description metadata values:

```python
>>> for key in ['titles', 'creators', 'descriptions']:
...     print res['metadata'][key]
...
[{u'title': u'RDA Foundation Governance Document'}]
[{u'creator_name': u'Research Data Alliance Council'}, {u'creator_name': u'RDA2'}]
[{u'description': u'A document describing the high-level structures of the Research Data Alliance Foundation. This document is separate from the regular governance document, which describes procedures and processes.', u'description_type': u'Abstract'}]
```

- Is the record open access and currently published?

#### Exercise 1c Download and compare checksum
Download a file from the record's file bucket and compare its checksum.

- Endpoint: `/api/files/<FILE_BUCKET_ID>/<FILE_KEY>`
- Method: GET
- Response: 200

Data:
- `FILE_BUCKET_ID`: 47077e3c4b9f4852a40709e338ad4620

Tasks:
- Determine the number of files stored in the record and store the file metadata in the variable `f`:

```python
>>> print len(res['files'])
1
>>> f = res['files'][0]
```

- Locate the file bucket ID for the file in the metadata and store it in the variable `bucket_id`:

```python
>>> print f['bucket']
2686d997-87e2-457f-996e-436bb55a84af
```

- Find the checksum of the file:

```python
>>> print f['checksum']
md5:c8afdb36c52cf4727836669019e69222
```

- Download the file to the `download` folder:

```python
>>> with open(f['key'], 'wb') as fout:
...     rf = requests.get("https://trng-b2share.eudat.eu/api/files/%s/%s" % (f["bucket"], f["key"]))
...     fout.write(rf.content)
```

- Generate the checksum of the downloaded file using the `md5` package:

```python
>>> import md5
>>> md5 = md5.md5(rf.content).hexdigest()
>>> print md5
c8afdb36c52cf4727836669019e69222
```

- Compare to the checksum provided:

```python
>>> print md5 == f['checksum'][4:]
True
```

### Communities and metadata schemas
In this exercise the metadata and metadata schema of a community is retrieved and processed.

#### Exercise 2a Retrieve existing communities
Retrieve a list of all defined communities.

- Endpoint: `/api/communities`
- Method: GET
- Response: 200

Tasks:
- Retrieve the communities listing
- Determine the number of existing communities

#### Exercise 2b Retrieve the records of a community
Retrieve a list of all records published under the EUDAT community.

- Endpoint: `/api/records?q=community:<COMMUNITY_ID>`
- Method: GET
- Response: 200

Data:
- `COMMUNITY_ID`: `e9b9792e-79fb-4b07-b6b4-b9c2bd06d095`

Tasks:
- Determine the required parameters for the request (`q=community:<COMMUNITY_ID>`)

```python
>>> params = {'q': 'community:e9b9792e-79fb-4b07-b6b4-b9c2bd06d095'}
```

- Execute the request with correct URL:

```python
>>> r = requests.get('https://trng-b2share.eudat.eu/api/records', params=params)
>>> print r
<Response [200]>
>>> res = r.json()
```

- Determine the number of records published:

```python
>>> print res['hits']['total']
689
```

- Show the metadata of the first published record:

```python
>>> print res['hits']['hits'][0]
{
...
}
```

#### Exercise 2b Retrieve community metadata schema
Retrieve the metadata schema of the EUDAT community.

- Endpoint: `/api/communities/<COMMUNITY_ID>/schemas/last`
- Method: GET
- Response: 200

Tasks:
- Retrieve the community metadata schema:

```python
>>> r = requests.get('https://trng-b2share.eudat.eu/api/communities/e9b9792e-79fb-4b07-b6b4-b9c2bd06d095/schemas/last')
>>> print r
<Response [200]>
>>> res = r.json()
```

- Store the schema in the variable `schema`:

```python
>>> schema = res['json_schema']['allOf'][0]
```

#### Exercise 2c Investigate metadata schema structure
Determine the required metadata fields and investigate the required structure of each field.

Use the result of the previous exercise.

Tasks:
- Determine the number of metadata schema fields defined:

```python
>>> print len(schema['properties'])
24
```

- Display all metadata fields defined in the schema:

```python
>>> print schema['properties'].keys()
[u'embargo_date', u'contributors', u'community', u'titles', u'descriptions', u'keywords', u'$schema', u'open_access', u'alternate_identifiers', u'_files', u'version', u'_deposit', u'disciplines', u'publisher', u'_oai', u'contact_email', u'publication_date', u'community_specific', u'publication_state', u'resource_types', u'language', u'license', u'_pid', u'creators']
```

Please note: the `$schema`, `_files`, `_deposit`, `_oai` and `_pid` are not to be filled in.

- Determine the required metadata schema fields:

```python
>>> print schema['required']
[u'community', u'titles', u'open_access', u'publication_state', u'community_specific']
```

- Determine the structure of the author, title and description field:

```python
>>> print simplejson.dumps(schema['properties']['titles'], indent=4)
{
    "description": "The title(s) of the uploaded resource, or a name by which the resource is known.",
    "title": "Titles",
...
    "type": "array"
}
```

### Record creation and data upload
In this final exercise, a new draft record is created and prepared for final publication. This includes adding initial metadata, updating it through a patch and adding files to publish. Final step is to change its state from draft to published.

#### Exercise 3a Create a new draft record
Create a new draft record with some metadata values.

- Endpoint: `/api/records/`
- Method: POST
- Response: 201

Tasks:
- Prepare the header
- Prepare the payloads
- Get your token to send with the request
- Check your draft record ID
- Get the `publication_state` metadata value
- Get the file bucket ID

#### Exercise 3b Upload files
Add files to your newly created draft record.

- Endpoint: `/api/files/<FILE_BUCKET_ID>/<FILE_KEY>`
- Method: PUT
- Response: 200

Tasks:
- Set the header
- Open the file handle of the file to be uploaded
- Send the file

#### Exercise 3c Update and complete metadata
Create a JSON patch to update the current metadata values of the draft record.

- Endpoint: `/api/records/<RECORD_ID>/draft`
- Method: PATCH
- Response: 200

Tasks:
- Create a JSON patch from the old metadata
- Remove all existing fields that need to be preserved


#### Exercise 3d Publish record
Create a final JSON patch to publish your draft record and make it publicly available.

- Endpoint: `/api/records/<RECORD_ID>/draft`
- Method: PATCH
- Response: 200

Tasks:
- Create the simple JSON patch
- Set the header
- Do the request and check the `publication_state` metadata field
- Check the DOI and ePIC PID
- Check your publication on the website