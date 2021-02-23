# weather-pipeline

The goal of this project is to build a weather data pipeline on Google Cloud Platform that:
- [x] starts with a sensor from IoT, 
- [x] utilizes a message queue to receive and deliver data, 
- [x] leverages a serverless function to move the data to a data warehouse and then, 
- [x] create a dashboard that displays the information. 

## Pipeline Architecture ##

![image](https://github.com/ioantsep/weather-pipeline/pipeline.png)

![alt text](https://raw.githubusercontent.com/ioantsep/weather-pipeline/branch/path/to/img.png)

![Image of Yaktocat](https://octodex.github.com/images/pipeline.png)

![pipeline](https://user-images.githubusercontent.com/47212261/108886813-09eae080-75cf-11eb-8147-3271c50161d4.png)

## Dataset Information ##
- Data in JSON format from an IoT sensor (9 features)

## **Data Flow** ##
- __Cloud Storage:__ data from the sensor

- __Data Flow 1:__ send to Cloud Dataflow

- __Data Flow 2:__ send to Cloud Pub/Sub. The Cloud Dataflow is the Publisher for messages 

- __Data Flow 3:__ send to Cloud Functions. Every time that there is a message, the Cloud Functions is triggering and the message goes to Subscriber

- __Data Flow 4:__ send to BigQuery which is the  Subscriber for Pub/Sub and the data is stored

- __visualization__: using Data Studio



## **Build, Provision and Deploy the Project on GCP** ##
1. Sign-in to Google Cloud Platform console and create a new project called --> "iotpipeline".
1. Sign-in στην Google Cloud Platform: εισάγουμε τα απαραίτητα ζητούμενα στοιχεία του λογαριασμού μας για να εισέλθουμε στην κεντρική σελίδα της κονσόλας 
2. Δημιουργία νέου project: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "IAM & admin", έπειτα την καρτέλα "Manage resources" και δημιουργούμε ένα νέο έργο (project) με ονομασία "iotpipeline". Το αναγνωριστικό του έργου (ID) πρέπει να είναι ένα μοναδικό όνομα σε όλα τα έργα του Google Cloud, οπότε και το σημειώνουμε (ID: iotpipeline-243711).
3. Δημιουργία πίνακα, που θα περιέχει τα δεδομένα, στη BigQuery: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "BigQuery", που βρίσκεται στο θεματικό πεδίο "BIG DATA", κλικάρουμε πάνω στο projectID και επιλέγουμε "CREATE DATASET", όπου και συμπληρώνουμε το DatasetID, το οποίο είναι "weatherData". Συνεχίζουμε κλικάροντας το "CREATE TABLE", όπου και επιλέγουμε για το "Source Data" το "Empty table", για το "Table type" το "Native table", για το "Table name" το "weatherDataTable" και κάτω από το πεδίο "Schema", πατάμε στο "+Add field", όπου θα πρέπει να δημιουργήσουμε 9 πεδία με τον κατάλληλο τύπο. Έτσι έχουμε πλέον έτοιμη την αποθήκη δεδομένων για να δεχτεί τα δεδομένα μας.
4. Δημιουργία ενός Pub/Sub θέματος (topic): από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "Pub/Sub", που βρίσκεται στο θεματικό πεδίο "BIG DATA", κλικάρουμε πάνω στο "Topics", πατάμε στο "Enable API", πατάμε στο "Create a topic", όπου και συμπληρώνουμε το όνομα "weatherdata" και τέλος κλικάροντας στο "CREATE" δημιουργούμε το Pub/Sub topic μας. 
5. Χρήση Cloud Functions: στο σύστημα μας μια λειτουργία νέφους θα ξεκινά κάθε φορά που ένα μήνυμα θα δημοσιεύεται στο weather topic, θα διαβάσει το μήνυμα και στη συνέχεια θα το αποθηκεύσει στο BigQuery. Από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "Cloud Functions", που βρίσκεται στο θεματικό πεδίο "COMPUTE", πατάμε στο "Enable API", πατάμε στο "Create function", όπου συμπληρώνουμε τo πεδίo "name" με το "weatherPubSubToBQ", και επιλέγουμε ως Trigger το "Cloud Pub/Sub", ως Topic το "weatherdata" και ως Source code το "Inline editor". Στην καρτέλα "index.js", γράφουμε τον παρακάτω κώδικα JavaScript (Node.js 6): 
/**
 * Background Cloud Function to be triggered by PubSub.
 *
 * @param {object} event The Cloud Functions event.
 * @param {function} callback The callback function.
 */
exports.subscribe = function (event, callback) {
  const BigQuery = require('@google-cloud/bigquery');
  const projectId = "iotpipeline-243711"; 
  const datasetId = "weatherData"; 
  const tableId = "weatherDataTable"; 
  const PubSubMessage = event.data;
  // Incoming data is in JSON format
  const incomingData = PubSubMessage.data ? Buffer.from(PubSubMessage.data, 'base64').toString() : "{'sensorID':'na','timecollected':'1/1/1970 00:00:00','zipcode':'00000','latitude':'0.0','longitude':'0.0','temperature':'-273','humidity':'-1','dewpoint':'-273','pressure':'0'}";
  const jsonData = JSON.parse(incomingData);
  var rows = [jsonData];

  console.log(`Uploading data: ${JSON.stringify(rows)}`);

  // Instantiates a client
  const bigquery = BigQuery({
    projectId: projectId
  });

  // Inserts data into a table
  bigquery
    .dataset(datasetId)
    .table(tableId)
    .insert(rows)
    .then((foundErrors) => {
      rows.forEach((row) => console.log('Inserted: ', row));

      if (foundErrors && foundErrors.insertErrors != undefined) {
        foundErrors.forEach((err) => {
            console.log('Error: ', err);
        })
      }
    })
    .catch((err) => {
      console.error('ERROR:', err);
    });
  // [END bigquery_insert_stream]


  callback();
}; 

	ενώ στην καρτέλα "package.json", γράφουμε τον παρακάτω κώδικα:

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

	Τέλος στο πεδίο "Function to execute" γράφουμε "subscribe" και πατάμε το κουμπί "Create" για να δημιουργηθεί η συνάρτηση. Μόλις δημιουργηθεί, έχουμε καταφέρει να συνδέσουμε την υπηρεσία Pub/Sub με τη BigQuery.
	
6. Δημιουργία ενός αποθηκευτικού κουβά (storage bucket) που θα χρησιμεύσει ως μια προσωρινή αποθήκη για την υπηρεσία Dataflow: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "Storage", που βρίσκεται στο θεματικό πεδίο "STORAGE", κλικάρουμε πάνω στο "Browser" και στη συνέχεια πατάμε "Create bucket". Επιλέγουμε ένα μοναδικό όνομα, το "iotpipeline-bucket" και πατάμε "Create".
7. Ενεργοποίηση του Dataflow API: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "API & Services", πατάμε στο "Enable API and Services" και στη γραμμή αναζήτησης του "Welcome to the new API Library", γράφουμε "Dataflow" και στη συνέχεια πατάμε στο " Google Dataflow API" και στο τέλος πατάμε το "Enable".
8. Δημιουργία εργασίας από πρότυπο (template) στο Dataflow: από το μενού πλοήγησης πάνω αριστερά, επιλέγουμε την καρτέλα "Dataflow", που βρίσκεται στο θεματικό πεδίο "BIG DATA", κλικάρουμε πάνω στο "CREATE JOB FROM TEMPLATE" και στη συνέχεια συμπληρώνουμε το "Job name" με "dataflow-gcs-to-pubsub4", το "Cloud Dataflow template" με "Text Files on Cloud Storage to Cloud Pub/Sub", το "Input Cloud Storage File(s)" με "gs://codelab-iot-data-pipeline-sampleweatherdata/*.json" (public dataset), το "Output Pub/Sub Topic" με "projects/iotpipeline-243711/topics/weatherdata" και το "Temporary location" με "gs://iotpipeline-bucket/tmp". Πατάμε στο κουμπί "Run job" και αρχίζει η εργασία του Dataflow.
9. Έλεγχος αν υπάρχει ροή δεδομένων και αν αυτά τοποθετούνται στον πίνακα που δημιουργήσαμε στη BiqQuery στο βήμα 3: ανοίγουμε την καρτέλα BigQuery, επιλέγουμε to project (iotpipeline-243711), το Dataset (weatherData), και τον πίνακα (weatherDataTable) και κλικάρουμε στο "QUERY TABLE", όπου στον Query editor, πληκτρολογούμε την εντολή (μορφής SQL): 			  SELECT * FROM `iotpipeline-243711.weatherData.weatherDataTable` LIMIT 1000										  όπου εμφανίζει το μια σελίδα με το "Query results", το οποίο περιέχει τις μέχρι εκείνη τη στιγμή εγγραφές με όριο τις 1000.
10. Δημιουργία γραφημάτων με το Data Studio: από τη σελίδα με το "Query results", πατάμε στο "EXPLORE WITH DATA STUDIO", ή ανοίγουμε μια ξεχωριστή σελίδα στον φυλλομετρήτη στη διεύθυνση https://datastudio.google.com, και από εκεί πλέον οπτικοποιούμε τα δεδομένα, δημιουργώντας εκθέσεις (reports) επιλέγοντας πολλών ειδών διαγράμματα και πίνακες.



```
git status
git add
git commit
```
