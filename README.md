# Data Engineering
## Convert CSV file to JSON file using (nifi)

The main Idea of our exercise is to convert comma-separated value (CSV) format to JavaScript Object Notation (JSON) format by using Apache NiFi.

# Methodology

- Prepare the environment by using the docker-compose file ( attached in the repository) images (jupyter/minimal-notebook,mkobit/nifi,and bitnami/zookeeper).
- Create CSV file by (jupyter).
- Convert the CSV file to JSON by (Nifi).

## 1. Prepare the environment by using the docker-compose file

After building the docker-compose file, use the below command:
```sh
docker compose up
```
| App | Link |
| ------ | ------ |
| JupyterLab | [http://localhost:8888/]|
| Nifi| [http://localhost:8080/]|

| Directory | Des | Path |
| ------ | -------  | ------ |
| Input | CSV file |/home/csv_data|
| Output| JSON file |/home/json_data|

## 2. Create CSV file by (jupyter):
using jupyter notebook to create CSV file by using faker package "generates fake data for you" check the below code:
![code1](https://user-images.githubusercontent.com/102326351/170957435-e31f9fef-973b-427b-9627-a9dbd9564f7b.PNG)
![code2](https://user-images.githubusercontent.com/102326351/170957804-fbfd27d4-65a8-4935-8f7e-0a10e77f9288.PNG)

## 3. Convert the CSV file to JSON by (Nifi):
The below picture illustrates our framework within nifi :
![NIFI](https://user-images.githubusercontent.com/102326351/170964015-75d14b4f-755c-44b6-a220-7a956d113858.PNG)

### Get CSV file :
Add Getfile Processor with the following properties:

- Input Directory : home/csv_data
- file Filter : '[^\.].*.csv
- Keep Source file: true ( if you need).

![Get file](https://user-images.githubusercontent.com/102326351/170966634-7fe21c32-9f64-45a4-ae0b-0e41a26ec7e7.PNG)

### Apply schema on CSV file:
Since I will use AvroSchema on the CSV file to convert it to JSON, I will add UpdateAttribute  processor to add "schema.name" property:

- schema.name : users ( you can choose any name related to your data)

![schema name](https://user-images.githubusercontent.com/102326351/170968433-1a798073-f274-4e9e-aed7-5ccb119bd909.PNG)

### Convert to JSON file:

1. Add ConvertRecord Processor with the following properties:

| Property | Value |
| ------ | ------ |
| Record Reader |CSVReader|
| Record Writer|JsonRecordSetWriter|


![ConvertRecord](https://user-images.githubusercontent.com/102326351/170970601-88642888-0d4a-4174-9d55-7cae4d88293c.PNG)

2. We need to provide a schema for our data by adding AvroSchemaRegistry by clicking on the arrow highlighted in the previous picture:

![avroschema](https://user-images.githubusercontent.com/102326351/170972018-6ad1c641-3366-4a33-8d75-8e624147d8e1.PNG)

3. we need to configure each property by clicking on the higlighted icons for each one

### -AvroSchemaRegistry: the same name of schema that you used before. 

#### users schema:

```sh
{
  "type": "record",
  "name": "UserRecord",
"fields" : [
    {"name": "name", "type": ["null", "string"]},
    {"name": "age", "type": ["null", "string"]},
    {"name": "street", "type": ["null", "string"]},
    {"name": "city", "type": ["null", "string"]},
    {"name": "state", "type": ["null", "string"]},
    {"name": "zip", "type": ["null", "string"]},
    {"name": "lng", "type": ["null", "string"]},
    {"name": "lat", "type": ["null", "string"]}
  ]
}|
```
![avro configuer](https://user-images.githubusercontent.com/102326351/170973343-804fba4d-89d9-4388-b8cb-f98f687dfefa.PNG)

### -CSVReader: 

| Property | Value |
| ------ | ------ |
| Schema Access Strategy |Use 'Schema Name' Property|
| Schema Registry|AvroSchemaRegistry_users|
| Schema Name|${schema.name}|

![csv reader confi](https://user-images.githubusercontent.com/102326351/170975002-685e194e-38a4-4557-a188-58887753d2b6.PNG)

### -JsonRecordSetWriter: 

| Property | Value |
| ------ | ------ |
| Schema Write Strategy |Set 'avro.schema' Attribute|
| Schema Access Strategy|Use 'Schema Name' Property|
| Schema Registry|AvroSchemaRegistry_users|
| Schema Name|${schema.name}|

![Json writer conf](https://user-images.githubusercontent.com/102326351/170975535-1609e741-600b-4b0b-a28f-024175c981f4.PNG)


### Rename file with JSON extension :
Rename the file with JSON extension by adding UpdateAttribute processor with the following properties :

| Property | Value |
| ------ | ------ |
| filename |${filename:substringBeforeLast('.')}.json|

![rename file](https://user-images.githubusercontent.com/102326351/170977459-d470f351-764b-48ad-912e-09e292b43785.PNG)


### Put the JSON file in the output path :
Add the PutFile processor with the following properties :

| Property | Value |
| ------ | ------ |
| Directory |/home/json_data|
| Permissions |-rwxrwxrwx|

![put file](https://user-images.githubusercontent.com/102326351/170978379-d9415973-23b2-406e-88f4-277aed4d60cb.PNG)


# Pipeline :

![Pipeline](https://user-images.githubusercontent.com/102326351/170978717-fbe2e523-3ddf-43ad-a7a3-68f38e326afe.PNG)
