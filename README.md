# weather-pipeline-on-Google-Cloud-Platform

The goal of this project is to build a weather data pipeline on Google Cloud Platform that:
- [x] starts with a sensor from IoT, 
- [x] utilizes a message queue to receive and deliver data, 
- [x] leverages a serverless function to move the data to a data warehouse and then, 
- [x] create a dashboard that displays the information. 

## Dataset Information ##
- Data in JSON format from an IoT sensor (9 features)



## Pipeline Architecture ##

![pipeline](https://github.com/ioantsep/weather-pipeline-on-GCP/blob/main/Images/pipeline.png)



## **Data Flow** ##
- __Cloud Storage:__ data from the sensor

- __Data Flow 1:__ send to Cloud Dataflow

- __Data Flow 2:__ send to Cloud Pub/Sub. The Cloud Dataflow is the Publisher for messages 

- __Data Flow 3:__ send to Cloud Functions. Every time that there is a message, the Cloud Functions is triggering and the message goes to Subscriber

- __Data Flow 4:__ send to BigQuery which is the  Subscriber for Pub/Sub and the data is stored

- __visualization__: using Data Studio


## **Build, Provision and Deploy the Project on GCP** ##
1. Sign-in to Google Cloud Platform console and create a new project, project_name = "iotpipeline", project_ID = "iotpipeline-243711'".

2. Creation of a table in BigQuery: "BIG DATA" --> "BigQuery" --> click on projectID --> "CREATE DATASET" with DatasetID = "weatherData" -->  click on "CREATE   TABLE" --> "Source Data" --> "Empty table", "Table type" = "Native table", "Table name" = "weatherDataTable", "Schema" --> "Add field" with 9 features. 

3. Creation of a Pub/Sub topic: "BIG DATA" --> "Pub/Sub" --> "Topics" --> "Enable API" --> "Create a topic", name ="weatherdata" --> "CREATE". 

4. Connect Pub/Sub with BigQuery using Cloud Functions: "COMPUTE" --> "Cloud Functions" --> "Enable API" --> "Create function", "name" = "weatherPubSubToBQ", "Trigger" = "Cloud Pub/Sub", "Topic" = "weatherdata", "Source code" = "Inline editor". In tab "index.js", write the JavaScript code (Node.js 6): [index.js](https://github.com/ioantsep/weather-pipeline/blob/main/index.js) and in the tab "package.json" write code : 	[package.json](https://github.com/ioantsep/weather-pipeline/blob/main/package.json). Then in "Function to execute" = "subscribe" --> "Create".	
	
5. Creation of a storage bucket for Dataflow: "STORAGE" --> "Browser" --> "Create bucket", "iotpipeline-bucket" --> "Create".

6. Dataflow API: "API & Services" --> "Enable API and Services" --> "Welcome to the new API Library", search bar = "Dataflow" --> " Google Dataflow API" --> "Enable".

7. Creation of template in Dataflow: "BIG DATA" --> "Dataflow" --> "CREATE JOB FROM TEMPLATE" with:
 - "Job name" = "dataflow-gcs-to-pubsub4", 
 - "Cloud Dataflow template" = "Text Files on Cloud Storage to Cloud Pub/Sub",
 - "Input Cloud Storage File(s)" με "gs://codelab-iot-data-pipeline-sampleweatherdata/*.json" (public dataset), 
 - "Output Pub/Sub Topic" = "projects/iotpipeline-243711/topics/weatherdata", 
 - "Temporary location" = "gs://iotpipeline-bucket/tmp".
 
   Click on "Run job" and Dataflow starts.
   
   	![dataflow](https://github.com/ioantsep/weather-pipeline-on-GCP/blob/main/Images/dataflow.png)
   
   

8. Checking the Data Flow: BigQuery --> "iotpipeline-243711", Dataset = "weatherData", Table = "weatherDataTable" --> "QUERY TABLE", --> Query editor: 		
	```
	SELECT * FROM `iotpipeline-243711.weatherData.weatherDataTable` LIMIT 1000	
	```
	
	![BigQuery](https://github.com/ioantsep/weather-pipeline-on-GCP/blob/main/Images/BigQuery%20table.png)
	
   
	

9. Creation graphs using Data Studio: "Query results" --> "EXPLORE WITH DATA STUDIO".

	![DataStudio](https://github.com/ioantsep/weather-pipeline-on-GCP/blob/main/Images/Data%20Studio.png)
   

