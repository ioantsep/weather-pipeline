# weather-pipeline

The goal of this project is to build a weather data pipeline on Google Cloud Platform that:
- [x] starts with a sensor from IoT, 
- [x] utilizes a message queue to receive and deliver data, 
- [x] leverages a serverless function to move the data to a data warehouse and then, 
- [x] create a dashboard that displays the information. 

## Dataset Information ##
- Data in JSON format from an IoT sensor (9 features)



## Pipeline Architecture ##

![pipeline](https://github.com/ioantsep/weather-pipeline/blob/main/pipeline.png)



## **Data Flow** ##
- __Cloud Storage:__ data from the sensor

- __Data Flow 1:__ send to Cloud Dataflow

- __Data Flow 2:__ send to Cloud Pub/Sub. The Cloud Dataflow is the Publisher for messages 

- __Data Flow 3:__ send to Cloud Functions. Every time that there is a message, the Cloud Functions is triggering and the message goes to Subscriber

- __Data Flow 4:__ send to BigQuery which is the  Subscriber for Pub/Sub and the data is stored

- __visualization__: using Data Studio


## **Build, Provision and Deploy the Project on GCP** ##
1. Sign-in to Google Cloud Platform console and create a new project, project_name='iotpipeline', project_ID='iotpipeline-243711'.

2. Creation of a table in BigQuery: "BIG DATA" --> "BigQuery" --> click on projectID --> "CREATE DATASET" with DatasetID = "weatherData" -->  click on "CREATE   TABLE" --> "Source Data" --> "Empty table", "Table type" = "Native table", "Table name" = "weatherDataTable", "Schema" --> "Add field" with 9 features. 

3. Creation of a Pub/Sub topic: "BIG DATA" --> "Pub/Sub" --> "Topics" --> "Enable API" --> "Create a topic", name ="weatherdata" --> "CREATE" 

4. Connect Pub/Sub with BigQuery using Cloud Functions: "COMPUTE" --> "Cloud Functions" --> "Enable API" --> "Create function", "name" = "weatherPubSubToBQ", "Trigger" = "Cloud Pub/Sub", "Topic" = "weatherdata", "Source code" = "Inline editor". In tab "index.js", write the JavaScript code (Node.js 6):

	[index.js](https://github.com/ioantsep/weather-pipeline/blob/main/index.js)

and in the tab "package.json" write code :

	[package.json](https://github.com/ioantsep/weather-pipeline/blob/main/package.json)

{
  "name": "function-weatherPubSubToBQ",
  "version": "0.0.1",
  "private": true,
  "license": "Apache-2.0",
  "author": "Google Inc.",
  "dependencies": {
    "@google-cloud/bigquery": "^0.9.6"
  }
}
Then in "Function to execute" = "subscribe" --> "Create" Pub/Sub με τη BigQuery.	
	
6. Creation of a storage bucket for Dataflow: "STORAGE", --> "Browser" --> "Create bucket",  "iotpipeline-bucket" --> "Create".
7. Ενεργοποίηση του Dataflow API: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "API & Services", πατάμε στο "Enable API and Services" και στη γραμμή αναζήτησης του "Welcome to the new API Library", γράφουμε "Dataflow" και στη συνέχεια πατάμε στο " Google Dataflow API" και στο τέλος πατάμε το "Enable".
8. Δημιουργία εργασίας από πρότυπο (template) στο Dataflow: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "Dataflow", που βρίσκεται στο θεματικό πεδίο "BIG DATA", κλικάρουμε πάνω στο "CREATE JOB FROM TEMPLATE" και στη συνέχεια συμπληρώνουμε το "Job name" με "dataflow-gcs-to-pubsub4", το "Cloud Dataflow template" με "Text Files on Cloud Storage to Cloud Pub/Sub", το "Input Cloud Storage File(s)" με "gs://codelab-iot-data-pipeline-sampleweatherdata/*.json" (public dataset), το "Output Pub/Sub Topic" με "projects/iotpipeline-243711/topics/weatherdata" και το "Temporary location" με "gs://iotpipeline-bucket/tmp". Πατάμε στο κουμπί "Run job" και αρχίζει η εργασία του Dataflow.
9. Έλεγχος αν υπάρχει ροή δεδομένων και αν αυτά τοποθετούνται στον πίνακα που δημιουργήσαμε στη BiqQuery στο βήμα 3: ανοίγουμε την καρτέλα BigQuery, επιλέγουμε to project (iotpipeline-243711), το Dataset (weatherData), και τον πίνακα (weatherDataTable) και κλικάρουμε στο "QUERY TABLE", όπου στον Query editor, πληκτρολογούμε την εντολή (μορφής SQL): 			  SELECT * FROM `iotpipeline-243711.weatherData.weatherDataTable` LIMIT 1000										  όπου εμφανίζει το μια σελίδα με το "Query results", το οποίο περιέχει τις μέχρι εκείνη τη στιγμή εγγραφές με όριο τις 1000.
10. Creation graphs using Data Studio: από τη σελίδα με το "Query results", πατάμε στο "EXPLORE WITH DATA STUDIO", ή ανοίγουμε μια ξεχωριστή σελίδα στον φυλλομετρήτη στη διεύθυνση https://datastudio.google.com, και από εκεί πλέον οπτικοποιούμε τα δεδομένα, δημιουργώντας εκθέσεις (reports) επιλέγοντας πολλών ειδών διαγράμματα και πίνακες.



```
git status
git add
git commit
```
