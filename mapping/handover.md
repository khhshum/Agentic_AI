# Mapping Directory

## 1.	all_field.csv (must prepared)
    -Store all the Elasticsearch Database Field, Explanation, Calculation Method and Data Extraction Method, FieldSetID
    
    -Field: the field names of the database

    -Explanation: Meaning of the fields(e.g. What the field represents/ Device status (up=1 , down= 0))
	** Specified filter condition must include and specified the unit **
    
    -Calculation Method and Data Extraction Methodd: How to use it for calculation(e.g. use serial_diff, cardinality, max)
    
    -FieldSetID: 0=commonly used, 1=Huawei, 2=Aruba (Can be replaced by {async def get_filtered_csv() -> str:} from tool.py)

## 2. customer.xlsx (must prepared)
    -Store all the customer names in it for identifying the customer

## 3. wifi_customer_brand.xlsx (Not necessary)
    -Store the customer wifi brands. (Can be replaced by using  {async def get_filtered_csv() -> str:} from tool.py)

## 4. filter_field_description.csv (No need to prepared this)
    -End result of all_field_csv filtered by the index mapping.

## 5.flattened_mapping.json (No need to prepared this)
    -Flattened the raw_mapping.json
    -Used for filter the all_field.csv
    -Used for field selection when finding mapping

#
##
#

# RAG Directory
### JSON used for RAG database

## 1. DSL_example_ARUBA_copy.json
    -Store the History report with the description of each report
    -Used for History mode 
    -Will be embedded into a ChromaDB for RAG retrieval
    -New correct report will be saved here

## 2. DSL_example_ARUBA.json
    -the previous version of DSL_example_ARUBA.json

## 3. DSL_example_HUAWEI_copy.json
    -Store the History report with the description of each report
    -Used for History mode
    -Will be embedded into a ChromaDB for RAG retrieval
    -New correct report will be saved here

## 4. DSL_example_HUAWEI.json
    -the previous version of DSL_example_HAWEI.json
    
#
##
#

# Result Directory
### Store all the result after each run
  

## 1. result.xlsx
    -this is the report

## 2. result.json
    -this is the result retrieve from EMS Portal

## 3. result.py
    -the python code including calling API and DSL query

## 4. DSLQuery.json
    -the Elasticsearch Query for retrieving data

#
##
#

# Tools
## 1. Get_keyword_tool
    - Input the correct field name and customer name.
    - Then it will call API and retrieve all the possible value of that field
    - It is used for finding valid filter conditions for filter_finder_agent

## 2. Get_mapping_tool
    - Input the customer name 
    - Then it will call API to retrieve the index mapping of that customer
    - It will then call the helper functions {flatten_es_mapping} to help flattened the mapping 
    - It return the flattened mapping and save it as {flattened_mapping.json}

## 3. Get_csv_tool
    - It retrieve the all_fields.csv from mapping directory
    - It used multiple helper function to filter the csv file. 
    - It filter it base on the flatten mapping, ensuring all fields on the csv exist in the mapping.
    - Return the it as string and save it as {filter_field_description.csv}

## 4. Opendistro_search_tool
    - Input the customer name, the Elasticsearch query(DSL code)
    - It call the API to EMS portal and retrieve the data
    - It will save the result as {result.json}

## 5. Get_time_tool
    - It retrieve the current time, date and timezone.

#
##
#

# Requirements Team
### Summarization: The team will ask users questions to retrieve the correct and clear user requirements
## 1. customer_finder_agent
    -It will identify the customer base on the user input and retrieve the mappping
    -It will ask if not found

## 2. filter_finder_agent
    -It will identify what kind of filter need to be implemented
    -It is equip with 3 tools [Get_csv_tool, Get_keyword_tool,Get_time_tool]
    -It will ask and assure correct filter is implemented

## 3. requirements_analyzer
    -It will summarize all the requirements and output in the format below

 ```json
        {
        "customer": "",
        "summary": "",
        "mentioned_metrics": [],
        "mentioned_filters": [],
        "mentioned_groupings": [],
        "mentioned_time_range": [],
        "mentioned_methodology": []
        }
```


#
##
#

# ES_Query (Fast Mode)

### Modification:  ``` load_field_metadata```, ```load_database_schema```,``` _extract_schema_fields```,``` _filter_fields_by_schema ``` all these functions are used to find the filter_field_description.csv and flatten_mapping.json. This is uneseccary since it is saved in mapping directory. Suggest to remove these functions and input correct files instead. 
#
### Summarization: This team will recieve the user requirements and select correct fields then generate the workable Elasticsearch Query(DSL)
## 1. extract_query_fields
    - This AI Agent will choose fields base on the summarization of the requirements team and the csv. 
    - It will look through the csv and choose it base on the logic of the csv. 
    - It will output a structured output of :
    {
        filtering_fields:[]
        grouping_fields:[]
        metric_fields:{}
    }

## 2. generate_elasticsearch_query
    - Base on the output of the last Agent, it will generate correct Elasticsearch Query(DSL)


#
##
#
# Config

### Easier for handling files and 

## 1.get_model_client
    -Get the GPT-4o model, all API key, endpoint and api version are hiden from the py files(saved in model_config.ymal)
    -Allow user to set temperature

## 2. get_mini_model_client
    - same as above but for gpt-4o-mini instead of gpt-4o

## 3. get_embedding
    - Get the azure text-embedding-3-small
    - This model is used for RAG

## 4. File path
 ```team_state_dir```
 ```rag_dir```
 ```result_dir```
 ```mapping_dir```
 : Store all the folder path


#
##
#

# ReportSaverTeam
## 1. ReportSaver
    - This agent will only activated when being clicked on
