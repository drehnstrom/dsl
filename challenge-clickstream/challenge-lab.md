# DSL Challenge Lab: Analyzing Click Stream Data

## Understanding the Data

Sample data is located in the following Cloud Storage bucket. You should copy this data to a bucket in your own Google Coud project. 

```
gs://<bucket-to-be-determined>
```

The data represents visits to a web site. Each visit contains fields related to a user sessions including: Session ID, User ID, Device Type, ect. Visits also contain a collection of events and their related fields. There are three event types: Page View, Add Item to Cart, and Purchase. 

The OpenAPI schema for the data is as follows. Take a few minutes to familiarize yourself with this schema.

```
openapi: 3.0.0
info:
  title: Visit Schema API
  version: 1.0.0
  description: Schema for representing a visit to a website, including page views, adding items to a cart, and purchases.
paths: {}
components:
  schemas:
    Visit:
      type: object
      properties:
        session_id:
          type: string
          example: "SID-1234"
          description: "A unique identifier for the user's session."
        user_id:
          type: string
          example: "UID-5678"
          description: "A unique identifier for the user visiting the website."
        device_type:
          type: string
          enum: [desktop, mobile, tablet]
          example: "desktop"
          description: "The type of device used by the user."
        geolocation:
          type: string
          example: "37.7749,-122.4194"
          description: "The geolocation of the user in latitude,longitude format."
        user_agent:
          type: string
          example: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
          description: "The user agent string of the browser/device used by the user."
        events:
          type: array
          items:
            $ref: '#/components/schemas/Event'
          description: "List of events during the user's visit."

    Event:
      type: object
      properties:
        event_type:
          type: string
          enum: [page_view, add_item_to_cart, purchase]
          example: "page_view"
          description: "The type of event that occurred."
        timestamp:
          type: string
          format: date-time
          example: "2023-08-10T12:34:56Z"
          description: "The exact time when the event occurred."
        details:
          type: object
          oneOf:
            - $ref: '#/components/schemas/PageViewDetails'
            - $ref: '#/components/schemas/AddItemToCartDetails'
            - $ref: '#/components/schemas/PurchaseDetails'
          description: "Specific details of the event based on its type."

    PageViewDetails:
      type: object
      properties:
        page_url:
          type: string
          example: "https://example.com/products"
          description: "The URL of the webpage that was viewed."
        referrer_url:
          type: string
          nullable: true
          example: "https://google.com"
          description: "The URL of the referrer page that led to this page view, or null if none."

    AddItemToCartDetails:
      type: object
      properties:
        product_id:
          type: string
          example: "HDW-001"
          description: "The unique identifier of the product added to the cart."
        product_name:
          type: string
          example: "Laptop X200"
          description: "The name of the product added to the cart."
        category:
          type: string
          enum: [hardware, software, peripherals]
          example: "hardware"
          description: "The category of the product added to the cart."
        price:
          type: number
          format: float
          example: 999.99
          description: "The price of the product added to the cart."
        quantity:
          type: integer
          example: 2
          description: "The quantity of the product added to the cart."

    PurchaseDetails:
      type: object
      properties:
        order_id:
          type: string
          example: "ORD-4321"
          description: "A unique identifier for the order."
        amount:
          type: number
          format: float
          example: 1999.98
          description: "The total amount of the purchase."
        currency:
          type: string
          example: "USD"
          description: "The currency used for the purchase."
        items:
          type: array
          items:
            $ref: '#/components/schemas/PurchaseItem'
          description: "A list of items purchased in this order."

    PurchaseItem:
      type: object
      properties:
        product_id:
          type: string
          example: "HDW-001"
          description: "The unique identifier of the product purchased."
        product_name:
          type: string
          example: "Laptop X200"
          description: "The name of the product purchased."
        category:
          type: string
          enum: [hardware, software, peripherals]
          example: "hardware"
          description: "The category of the product purchased."
        price:
          type: number
          format: float
          example: 999.99
          description: "The price of the product purchased."
        quantity:
          type: integer
          example: 2
          description: "The quantity of the product purchased."
```

## Task 1: Migrating the Data to BigQuery

Your task is to not only migrate the data to BigQuery, but also to modify the schema so it is easy for Data Analysts to query. You could simply run an import with the Auto Detect Schema flag enabled. While this would work, it would be hard for most analysts to query the nested fields using SQL. 

1. Use the ititial schema as a starting point. Re-design the data so it would be more optimized for a BigQuery data warehouse. Draw a diagram of your planned schema changes. 

2. Write a program to migrate the data into BigQuery. You can do this using Bash, Python, Java, or Dataform, but you need to write a program so it could be run repeatedly. You can use the BigQuery console to help, but the final results need to be code. Create a Dataflow Workbench instance and write the code in Jupyter Notebooks if you like. 

3. Using Looker Studio, create a dashboard that contains the following information: 
   1. Visits by Page
   2. Most Popular Items
   3. Sales by Category
   4. Visit by Device Type


## Task 2: Writing Dataflow Batch Pipelines

In this task, you need to use Apache Beam and Dataflow to run a batch processing pipeline to accomplish the same job as in the previous task. Read the data from Cloud Storage, parse it, and write it to BigQuery using a schema that is optimized for analytics. 

1. Using Apache Beam, create a Pipeline to migrate the Click Stream data to BigQuery in accordance with the schema you created earlier. Program the pipeline in a Jupyter Notebook. 

2. Once you have the pipeline tested, run it using Google Cloud Dataflow. 

## Task 3: Processing the Data in Real Time

In this task, you will use Google tools to process data in real time. The data is sent to a Pub/Sub topic. You will program subscribers to process the messages as they arrive. 

1. Write a simulator that creates visits and posts them as messages to a Google Pub/Sub topic. Use variables to control the number of visits sent per minute and how long the simulator should run. You can program this any way you like. 

2. Create a Push subscriber running in Cloud Run or Cloud Functions. Process the messages as they come in. Parse them, and write the data to BigQuery where it can be analyzed. In Looker Studio, create a simple report that shows clicks by page in real time. 

3. Do the same thing as the previous step, but program a Pull subscriber. Deploy the program to a Compute Engine Virtual Machine. Use an Instance Group to set up autoscaling. Also, implement some kind of health check that you can use to ensure if the pull process is not running, the machine will be restarted. 

4. Write an Apache Beam pipeline with the following requirements:
   1. Write the raw data to files in Google Cloud Storage at regular intervals. 
   2. Parse the messages and write the data to BigQuery. 
   3. Calculate page views by minute. Create a Dashboard that reports this information.
   4. Run the pipeline in Dataflow. 

5. You want to detect a potential denial of service attack. Create an Apache Beam pipeline that calculates page views per minute. Log this information the Google Cloud logs. Create a log metric that reports this information in a Logging and Monitoring Dashboard. Next, create a log alert that triggers beyond some threshold. When the alert triggers, send yourself an email. 

6. Restart your Pub/Sub message simulator so enough messages are sent to trigger the alert. 

## Using Google Cloud Composer to Orchestrate Data Engineering Tasks

In this task, you will use Apache Airflow and Google Cloud Composer to automate a data engineering task. 

1. Create a Composer pipeline with the following requirements:
   1. When a file containing Click Stream data is written to a Cloud Storage bucket, trigger the pipeline. 
   2. Run a Dataflow job that processes the file, parses the data, and writes it to BigQuery. 
   3. After the Dataflow job completes, send a message to Pub/Sub indicating the file was processed. 

2. Create a subscriber to the Pub/Sub topic that notifies you that the file was succesfully processed. 


## Using Dataplex to Share Enterprise Data

In  this task, you will use Google Cloud Dataplex to share your click stream data with the organization. 

1. Using Datapplex, create a Data Mesh architecture to share your Click Stream data with the organization. The architecture has the following requirements:
   1. Zones for raw and curated data. 
   2. The raw zone contains the original JSON data contining the Click Stream events 
   3. The data should be automatically processed and written to BigQuery tables. The BigQuery tables are in the curated zone. 
   4. Share the data using Dataplex security. 
   5. Add appropriate metadata to the curated datasets using Dataplex tags and tag templates. 
   6. Enable data lineage to tracj changes to data over time. 