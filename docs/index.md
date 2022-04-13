# What is Stream?

Cribl Stream is an observability pipeline tool living between any data source and any destination. These destinations can be systems of analysis (Splunk, Elastic, etc.) or systems of retention (S3 buckets, Data Lakes, etc.). Stream routes, reduces, replays and redacts data sources in flight.

# Basic Elements

Stream uses Sources, Destinations, Routes and Pipelines to move data coming from its sources, processes through its pipelines (group of functions) and send the results to one or many destinations in the format that is best for the environment and its use cases.

## This Labs has 3 parts:

**Part 1 – Introduction to Stream elements**

**Part 2 – Routes and Pipelines**

**Part 3 – Answering to Use Cases**

Access your Cribl Stream instance from the jump box provided using the Chrome web browser pointing to: [Cribl](http://10.233.36.248:9000)
Login as Admin with the password of Go2atc4labs!
 
 
## Part 1 – Introduction to Stream elements

### 1.  Review Sources and Destinations

Sources
LogStream can receive continuous data input from various Sources, including Splunk, HTTP, Elastic Beats, Kinesis, Kafka, TCP JSON, and many others.
Sources can be of a Push or Pull type where Push sources can forward data into Cribl Stream as any other receiving system (Syslog, Splunk, etc.). 
Data from these Sources is normally sent to a set of LogStream Workers through a load balancer. Some Sources, such as Splunk forwarders, have native load-balancing capabilities, so you should point these directly at LogStream.
Pull sources can interact with Stream via REST APIs or other methods available within Stream integrations. For sources not necessarily integrated with Stream Scripts may also be used to collect data from Pull sources.

Destinations
Stream can send data to multiple destinations. These destinations may have native interactions as in sources or being sent at the protocol level such as Syslog, TCP, TCP/Json etc.
There may be Streaming, No Streaming and Other Destinations. These destinations can have integrations (as in Sources) or 
Streaming Destination will forward a continuous flow of data out of Stream to systems or connoting points such as Splunk HEC
Non-Stream Destination wills send data to retention endpoints such as S3 buckets, Azure Blob Storage and others.
Other Destinations will serve special purposes within the data flow.
[Destinations}(https://docs.cribl.io/logstream/destinations)


- Select Data/Sources > Sources from the top menu
- Explore all sources available, filter at the bottom selected top menu for Collectors, Push, Pull, System and Internal sources.

![Cribl-Lab01-1](_images/Cribl-Lab01-1.png)

- Select Push > Syslog
Your sources are listed on the left and under Manage Syslog Sources you may configure several Syslog sources. These sources will act as a Syslog server receiving data from any syslog capable device sending data to any existing Syslog server (SyslogNG, Rsylog, etc.).

- Click on the pre-configured source in_syslog and observe the configuration components available for this Source. Note, on the left menu, the out of the box TLS capability.

![Cribl-Lab01-2](_images/Cribl-Lab01-2.png)

 
### 2.  Configure a Source
- From the top menu select Data/Sources > Sources then from Push select Syslog
- Click on Add New From the top right button
- Enter the following values: 
- Input ID: Syslog_Source
- Address: 0.0.0.0
- UDP Port: 9514
- TCP Port: 9514
- Click Save and wait until the Live Status becomes green. 

![Cribl-Lab01-3](_images/Cribl-Lab01-3.png)

Your source has been configured, in this case Syslog, you may now send data from devices (firewalls, routers, servers, etc.) to the IP address for your Worker Node(s) and start receiving data. 

*Note: no data has been stored on Cribl Stream.* 
 

### 3.  Configure a Splunk Destination
- Select Data/Sources > Destinations
- Click on Splunk Single Instance (Tile)
- Click on Add New From the top right button
- Enter the following values: 
- Output ID: Splunk_Lab
- Address: 10.253.33.249
- Port: 9997
- Backpressure behavior: Block
- Click Save and wait for the Live status to become green
- Click on the Splunk_Lab destination
- Select Test in the context top menu and for Select Sample chose syslog.log
- Click Run Test

Observe the Test Results confirming your sample data was sent and received by the configured destination (Splunk_Lab)

![Cribl-Lab01-4](_images/Cribl-Lab01-4.png)

### 4.  Configure an Elastic Destination
- Select Data/Sources > Destinations
- Click on Elasticsearch (Tile)
- Click on Add New From the top right button

**Enter the following values:**

- Output ID: Elasticsearch
- Bulk API URL*: 10.253.33.250
- Index: elastic_lab
- Type: _doc
- Authentication Enabled toggle set to Yes
- Authentication Method button set to Manual
- Username: admin
- Password: Go2atc4labs!
- Backpressure behavior: Block
- Click Save and wait for the Live status to become green
 
- Click on the Elasticsearch destination
- Select Test in the context top menu and for Select Sample chose syslog.log
- Click Run Test

Observe the Test Results confirming your sample data was sent and received by the configured destination (Elasticsearch).

![Cribl-Lab01-5](_images/Cribl-Lab01-5.png)

### 5.  Configure a S3 Bucket as Destination
 
For this task we will use an internal object storage solution to represent the S3 behavior or Simple Storage Service (Amazon S3). From the top menu select Data / Destinations. 

- From the list of integrations select the MinIO tile
- Click on Add New From the top right button

**Enter the following values:**

- Output ID: S3_Minio
- MinIO Endpoint*: http://192.168.2.52:9000
- MinIO Bucket Name*: 's3-syslog'
- Staging Location*: $CRIBL_HOME/state/outputs/staging
- Key Prefix*: Cribl
- Partitioning Expression: C.Time.strftime(_time ? _time : Date.now()/1000, '%Y/%m/%d')
- Data Format: json
- File Name Prefix Expression:  `CriblOut`
- File Name Suffix Expression:  `.${C.env["CRIBL_WORKER_ID"]}.${__format}${__compression === "gzip" ? ".gz" : ""}`
- Compress: none
- Backpressure behavior: Block
- Tags: <LEAVE_EMPTY>
- Click Save

![Cribl-Lab01-6](_images/Cribl-Lab01-6.png)

- You will be back on the Destinations list. Wait until the Status becomes green and click on S3_Minio
- On the top menu (in the context window) click Test
- On Select sample chose syslog.log and click Run Test
 
### 6.  Create a S3 Source
 
Now that we have a local S3 destination configured receiving data in your S3 bucket, lets configure a S3 collector to use the Replay function within Cribl Stream. 
We will configure a regular S3 bucket as a Source, the same way you would do if you wanted to read data from an AWS S3 bucket. 
 
- From the list of integrations select the Collectors S3 tile
- Click on Add New From the top right button

**Enter the following values:**

- Output ID: S3_collect
- Auto-populate from: <LEAVE_BLANK>
- S3 bucket*:  's3-syslog'
- Region: <LEAVE_BLANK>
- Path: /Cribl/${_time:%Y}/${_time:%m}/${_time:%d}/
- Extend AUTHENTICATION and Select Manual
- Access key: admin
- Secret key: Go2atc4labs!
- Click Save

- Back to the Sources list click on the S3_collect source configured
- On the bottom left click Run
- On the new context window click Run

On the result context window, you should see the content from the S3 destination (syslog.log sample file) played back as result

![Cribl-Lab01-7](_images/Cribl-Lab01-7.png)

## Quick Connect
Within Cribl Stream you can send data from sources to destinations already configured with a drag and drop action. You may also add a pipeline to your quick connection and process data independently of existing routes (if any configured)

### 7.  Use QuickConnect to send data to Splunk and Elastic
 
- From the top main menu select Routing/QuickConnect
- From Sources (left side) click +New Source
- From the new context window browse to System and Internal and mouse over Datagen then click ‘Select Existing’
- From the list of available datagen sources, select ‘syslog’
- On the new context window click Yes.
- Back on the Quick Connect panel click and drag the mouse connecting the Datagen/syslog (+ sign) to the Splunk Single Instance (SplunkCriblLab) connecting both objects.
 
### 8.  Apply a passthrough and 1 pipeline to the QuickConnect route.
 
A new context window will present you a selection to choose how to process the data in this connected route. 
- Click on Passthru (sending raw data to the destination, no pipelines applied) click save.

![Cribl-Lab01-8](_images/Cribl-Lab01-8.png)

### 9.  Analyze the results in Splunk or Elastic (Splunk and Elastic dashboards being created for richer visualization and value realization) 
 
- Mouse over the Single Instance (SplunkCriblLab) Destination and click ‘Capture’ to display if data is being sent to the selected destination. 
- From the new context window validate if syslog data (your datagen source in this case) is being sent to the Splunk Single Instance configured.

![Cribl-Lab01-9](_images/Cribl-Lab01-9.png)

Follow the same process to add Elasticsearch as a second destination receiving the same source (Datagen syslog) already sending data to Splunk Single Instance.

- Mouse over Elasticsearch and click on ‘Capture’ to validate if data is being sent to the configured destination.
- Select from the top menu Data/Sources and click the Datagen tile. 
- In the syslog datagen source, click on ‘Connected Destinations’ from the left menu.
- On the right pane click on ‘Send to Routes’
- From the new context window click Yes
- Click Save.
 
## Part 2 – Routes and Pipelines

Now we have successfully sent data via QuickConnect lets use Routes to send data to the same destinations. 
First let’s enable another Source from our Datagen (windows_xml) 
 
### 1.  Enable a new source

- From the top menu click on Data/Source
- Click on the Datagen tile 
- From the Manage Datagen Sources pane click on the no toggle under the Enable column on the windows_xml source.
- On the new context window click Yes.
- On the same source (windows_xml) click on Live under the Status column and validate the proper sources is being generated.
- Click on the top X on the opened context window. 
          
### 2.  Create a new Route

- From the top menu select Routing/Data Routes
- In the Routes panel (left) click the + Route to add a new Route to the existing ones. 

![Cribl-Lab01-10](_images/Cribl-Lab01-10.png)

**On the newly create Route enter the following values:**

- Route Name*: to_Splunk
- Filter:  select   __inputId=='datagen:windows_xml'
- Pipeline*: passthru
- Enable Expression: No
- Output: select   splunk:SplunkCriblLab
- Final: No

![Cribl-Lab01-11](_images/Cribl-Lab01-11.png)

- Click Save
 
### 3.  Capture sample data from the route created
 
- Make sure your new route is not below any other routes with the Final toggle set to yes, if it is drag it above that final route. 
- From the to_Splunk Route click on the most left 3 dots and select Capture

![Cribl-Lab01-12](_images/Cribl-Lab01-12.png)

On the new context window validate you are capturing samples from your configured Route

![Cribl-Lab01-13](_images/Cribl-Lab01-13.png)

- At the bottom right click on Save as Sample File

**On the new context window enter the following values:**

- File Name*: windows_xml_sample.log
- Description: <LEAVE_EMPTY>
- Expiration (hours): <LEAVE_EMPTY>
- Tags: <LEAVE_EMPTY>
 
- From the right pane validate if your sample file has been created (if not refresh your browser) 
- Under Preview click Simple an validate your sample data from your Route.

![Cribl-Lab01-14](_images/Cribl-Lab01-14.png)

### 4.  Create a Pipeline

Let’s create a simple pipeline to process our Syslog source.

- Go to the top menu and select Processing / Pipelines
- On the right pane click on the Syslog_sample.log sample file (we will use the captured sample with this Pipeline)
- On the left pane at the top click the ‘+ Pipeline’ button and select ‘Create Pipeline’

![Cribl-Lab01-15](_images/Cribl-Lab01-15.png)

- On the left pane, enter only the ID as Syslog_test and click Save.

### 5.  Add functions to the pipeline

- Click on the ‘+ Function’ button on the right most side within the newly create Pipeline

![Cribl-Lab01-16](_images/Cribl-Lab01-16.png)

- Mouse click Standard / Eval or type Eval on the mini search bar and click Eval. 
- In the Function values enter the word ‘message’ (no quotes) in the Remove Fields field and click Save.
- Make sure your Syslog_sample.log sample file is load on the right pane and validate the results by clicking on the OUT button on the top bar within the Sample data loaded.

![Cribl-Lab01-17](_images/Cribl-Lab01-17.png)

Now that we have excluded the field message from the processing logs, lets use another select and use another Function “Drop” and reduce our data even further.
- On the left pane click on the ‘+ Function’ button and select Standard/Drop or type Drop on the mini search bar and click on the result.
- With the Function loaded, enter the following values in the field Filter: appname==’itaque’

![Cribl-Lab01-18](_images/Cribl-Lab01-18.png)

*Note the events matching the filter in the Drop function are greyed out and not send to the destination reducing the number of events.*

Let’s add another function to change/redact our data

- From the top bar on the left pane click on the “+ Function” button and select Standard/Rename or type Rename in the mini search bar and click on the result. 
- In the Function click the “+ Add field” button
- Within the Rename fields group type  facilityName in the Current name and NEW_facility_Name in the New Name fields
- Click Save

Observe the results on the right pane with the syslog_sample.log file selected and the OUT button enabled

![Cribl-Lab01-19](_images/Cribl-Lab01-19.png)

Now we have defined a Pipeline we need to attach it to a Route.

### 6.  Add a Pipeline to the Route

- On the left pane within your Splunk_test Pipeline, click on the top left link “Attach to Route”

You will be brought to the Routes list. 
- Select the Syslog_to_Splunk Ruote (created by you earlier)
- In Pipeline select Splunk_test

### 7.  Apply the destination to the Route

Now lets apply the destination that will receive the process stream from this Route.
- From Output select splunk:SplunkCriblLab
- In Description enter: “Sending Syslog data to Splunk”
- Final toggle set to No
- Click Save

![Cribl-Lab01-20](_images/Cribl-Lab01-20.png)


### 8.  Analyze results on Splunk or Elastic (no dashboards provided for this part)


