# PSLAManager 

This is a prototype of the PSLAManager system, based on the paper [Changing the Face of Database Cloud Services with Personalized Service Level Agreements](http://myria.cs.washington.edu/publications/Ortiz_PSLA_CIDR_2015.pdf). The PSLAManager generates service level agreements for cloud data services. Instead of asking users to pick a desired number of instances of a cloud service, the PSLAManager shows to users what is possible with their data. The key idea of the PSLA is for a user to specify the schema of her data and basic statistics (e.g., base table cardinalities) and for the PSLAManager to show what types of queries the user can run and the performance of these queries with different configurations (which we call "tiers") of the service, each with a defined cost. The PSLAManager is designed for read-only workloads and data analysis services.

## Setup
*Make sure to have Java installed in your machine before running PSLAManager*

**Running on Windows with Visual Studio**

1. Clone the repository

2. Open the solution ```PSLAManager.sln``` in Visual Studio and build the project. This will generate the ```PSLAManager.exe``` in the solution path (e.g. ```PSLAManager/bin/Release```). See instructions on how to run ```PSLAManager.exe``` under the section **Running PSLAManager**.

**Running on OS X with Mono**

1. Clone the repository

2. Install [Mono](http://www.mono-project.com/)

3. To build the project, go to the ```PSLAManager``` folder and run the command ```xbuild /p:BuildWithMono="true" /p:Configuration=Release PSLAManager.sln```

4. Navigate to the release folder ```PSLAManager/bin/Release```. Here, you will find the generated executable ```PSLAManager.exe```. See instructions on how to run ```PSLAManager.exe``` under the section **Running PSLAManager**.


##  Running PSLAManager
When generating a PSLA, the program takes as input the following components:

* **Dataset Schema**: description of the user's dataset that PSLAManager will use to generate a PSLA

* **Training Data**: This data contains statistics for queries from an independent synthetic dataset. This serves as a model to predict query runtimes for each tier.

* **Testing Data**: Given the dataset schema input, PSLAManager will generate a set of queries. Each query is then represented by a feature vector of statistics in order to predict the runtime. The last feature of the feature vector is the real query runtime. This feature is optional. When present, the PSLAManager can use either real or predicted runtimes when generating a PSLA. Real runtimes serve to test the PSLA generation capabilities in the case of perfect query time prediction.

In order for users to understand the performance of queries at different configruations, the PSLAManager system first uses the **Dataset Schema** to automatically generate queries for the dataset. PSLAManager primarily focuses on generating "join queries" at different selectivities. 

For each query generated, the user needs to generate a feature vector containing information about the query. The selection of these features are left up to the user. Some features could include those from the database query optimizer (such as query plan features, estimated number of rows, estimated costs, etc).

The runtimes for these queries are then predicted by using these features for different configurations of  the service through the **Training Data** and the **Testing Data**. As a final step, PSLAManager uses algorithms to compress the generated set of queries before showing the final PSLA. 

#### Option #1 : Generate a PSLA for the Myria Service and TPC-H Example Dataset
With this option, the system uses its default training data, testing data, and schema. The default schema uses the [TPC-H Star Schema dataset](http://www.cs.umb.edu/~poneil/StarSchemaB.PDF). The training data corresponds to a synthetic dataset executed on different sized Myria clusters of 4, 6, 8 and 16 workers. [Myria](http://myria.cs.washington.edu/) is our parallel data management service.

Under the ```PSLAManager\PSLAFiles``` directory you are provided with all the necessary components:
  * **Dataset Schema**: the file ```SchemaDefinition.txt``` provides information about the TPC-H SSB schema. 
  * **Training/Testing Data**: Under the folder ```predictions_for_tiers```, we provide testing data for the different tiers of the Myria service based on the TPC-H SSB dataset. We also provide training data for each of these tiers on a separate synthetic dataset.

When running PSLAManager, first click on the executable (```PSLAManager.exe```) generated in the solution path. If you're running this using mono, you can run with the command ```mono PSLAManager.exe```.

Once the program is running, first select the "Generate a PSLA" option. Second, you will need to select between using "predicted runtimes" or "real runtimes (assuming perfect predictions)". The resulting PSLA will appear under ```PSLAManager\PSLAFiles\FinalPSLA.json```.

#### Option #2: Customize the Dataset Schema to Generate a Myria PSLA
In this option, you can define your own dataset to generate a PSLA for your data based on the Myria Cloud service. If you wish to generate a PSLA for an entirely different cloud service, see option #3 below.

When defining a new schema, the statistics for the testing data must be updated to reflect the new queries generated by PSLAManager (according to the algorithm, the number of generated queries will change depending on the number of tables, attributes, sizes, etc. described in the schema). To customize  PSLAManager, follow these steps:

* **Step 1: Edit the schema for your dataset** Under ```PSLAManager\PSLAFiles``` edit the ```SchemaDefinition.txt``` to reflect the new schema. Documentation on how to edit is in the file.

* **Step 2: Collect Statisics for Testing** For each tier (4_workers, 6_workers, 8_workers and 16_workers), you will also need to provide statistics for the queries generated from your custom dataset.  
    * **Supplementary Files** You will first need PSLAManager to generate the queries (from your custom dataset) which you will be collecting statistics from. To do this, Run ```PSLAManager.exe``` and select the "Generate queries" option. This will create a file under ```PSLAManager\PSLAFiles``` called ```SQLQueries-Generated.txt```.
    *  **Examples of Testing Statistics**  You should see examples of statistics collected under ```PSLAManager\PSLAFiles\predictions_for_tiers\prediction_for_tiers``` under a tier folder with a file called ```TESTING.arff```.
    
* **Step 3: Run PSLAManager** Before running PSLAManager to generate a PSLA for your dataset, please ensure you have the following components ready:
    * A custom dataset defined under ```PSLAManager\PSLAFiles\SchemaDefinition.txt```
    * Modified testing data for each tier defined under ```PSLAManager\PSLAFiles\predictions_for_tiers```

 Once ready, run the ```PSLAManager.exe``` (or if using mono,```mono PSLAManager.exe```). and select the "Generate a PSLA" option. The resulting PSLA will appear as ```PSLAManager\PSLAFiles\FinalPSLA.json```.

#### Option #3 : Customize the Schema, Tiers, and Testing/Training Datasets to Generate a PSLA for a different Cloud Service
In this option, you can customize the dataset, number of tiers, and testing/training data for an entirely different cloud service.

When defining a new schema, the statistics for the testing data must be updated to reflect on new queries generated by PSLAManager (according to the algorithm, the number of generated queries will change depending on the number of tables, attributes, sizes, etc. described in the schema). To customize PSLAManager, follow these steps:

* **Step 1: Edit the schema for your dataset** Under ```PSLAManager\PSLAFiles``` edit the ```SchemaDefinition.txt``` to reflect the new schema. Documentation on how to edit is in the file.

* **Step 2: Select New Tiers** This is optional, you can keep the default configurations used for the Myria service (4, 6, 8, and 16) or select your own. To do this, go to the ```PSLAManager\PSLAFiles\predictions_for_tiers``` folder and edit the ```tiers.txt``` file. Instructions are provided in the file.

* **Step 3: Collect Statisics for Training** You will need to update training data update the model based on the cloud service you are trying to learn from.

    * **Supplementary Files** To help with the training, we provide both the queries and the data for a synthetic benchmark. You must collect statistics about this dataset for each tier you have defined in Step 2. Regardless of the statistics collected, you must be able to provide real query runtimes for a set of queries in order to predict runtimes for the testing data. To help with this step, we provide these queries under ```PSLAManager\PSLAFiles\predictions_for_tiers\custom_files\SQLQueries-Synthetic.txt```. In this directory, we also provide the DDL for the synthetic dataset. You can download the synthetic data from the following S3 bucket: ```s3://synthetic-dataset```. 
    * **Examples for Training Statistics** Under ```PSLAManager\PSLAFiles\predictions_for_tiers\prediction_for_tiers``` you will see a directory for each tier as defined in Step 2. Inside each tier in this directory, there is a file called ```TRAINING.arff``` which provides an example of statistics collected for training. If you will be providing your own statistics, please ensure that the query vectors for training data and testing data match.
    
* **Step 4: Collect Statisics for Testing** For each tier, you will also need to provide statistcs for the queries generated from your custom dataset.  
    
    * **Supplementary Files** You will first need PSLAManager to generate the queries (from your custom dataset) which you will be collecting statistics from. To do this, Run ```PSLAManager.exe``` and select the "Generate queries" option. This will create a file under ```PSLAManager\PSLAFiles``` called ```SQLQueries-Generated.txt```.
    *  **Examples of Testing Statistics**  Similarly to the  training data, you should see examples of testing statistics under ```PSLAManager\PSLAFiles\predictions_for_tiers\prediction_for_tiers``` under a tier folder with a file called ```TESTING.arff```.
    * **Optional (assuming perfect predictions)**:  If you would like to provide real runtimes for these different tiers and skip predictions altogether, for each  tier simply provide a ```realtimes.txt``` file with real runtimes for each query generated.

* **Step 5: Run PSLAManager** Before running PSLAManager to generate a PSLA, please ensure you have the following components ready:
    * A custom dataset defined under ```PSLAManager\PSLAFiles\SchemaDefinition.txt```
    * Modified tiers under ```PSLAManager\PSLAFiles\predictions_for_tiers\tiers.txt```
    * Modified training/testing data for each tier defined under ```PSLAManager\PSLAFiles\predictions_for_tiers```

Once ready, run the ```PSLAManager.exe```(or if using mono,```mono PSLAManager.exe```) and select the "Generate a PSLA" option. The resulting PSLA will appear as ```PSLAManager\PSLAFiles\FinalPSLA.json```

For any questions, please email jortiz16@cs.washington.edu.

