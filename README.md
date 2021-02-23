# weather-pipeline
WeatherPipe
The goal of this project is to provide an analysis tool for the NEXRAD dataset hosted in S3. The tool uses MapReduce to do the analysis and can be used of fairly large datasets. The tool uses EMR so that it can be easily used by anyone regardless of access to a MapReduce cluster. The tool is designed to abstract away setting up Map Reduce, marshaling of the NEXRAD data into usable data structures, and running the job.
