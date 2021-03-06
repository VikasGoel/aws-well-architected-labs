---
date: 2020-08-06T10:00:02-04:00
chapter: false
weight: 999
hidden: FALSE
---

### Data Transfer View
This view will be used to create the main **Data Transfer Cost Analysis** dashboard page.

{{% notice note %}}
We recommend large customers with over 500 linked accounts, or more than $5M a month in invoiced cost, display 1 or 2 months previous data instead of 3. Modify the INTERVAL in the statements below to less than 3 months for improved performance.
{{% /notice %}}

- {{%expand "Click here - to expand data transfer view query OR" %}}

Modify the following SQL query for data_transfer_view: 
- Update line 21 replace (database).(tablename) with your CUR database and table name 
- Optional: Adjust the look back from '3' months to desired time-frame in row 32,33

	    CREATE OR REPLACE VIEW data_transfer_view AS
            SELECT DISTINCT "line_item_product_code" "product_code" ,
                     "bill_billing_period_start_date" "billing_period" ,
                     "bill_payer_account_id" "payer_account_id" ,
                     "line_item_usage_account_id" "linked_account_id" ,
                     "product_product_name" "product_name" ,
                     "line_item_line_item_type" "charge_type" ,
                     "line_item_operation" "operation" ,
                     "product_region" "region" ,
                     "line_item_usage_type" "usage_type" ,
                     "product_from_location" "from_location" ,
                     "product_to_location" "to_location" ,
                     "line_item_resource_id" "resource_id" ,
                     ("sum"((CASE
                WHEN ("line_item_line_item_type" = 'Usage') THEN
                "line_item_usage_amount"
                ELSE 0 END)) / 1024) "TBs" , "sum"((CASE
                WHEN ("line_item_line_item_type" = 'Usage') THEN
                "line_item_usage_amount"
                ELSE 0 END)) "usage_quantity" , "sum"("line_item_blended_cost") "blended_cost" , "sum"("line_item_unblended_cost") "unblended_cost" , "sum"("pricing_public_on_demand_cost") "public_cost" , "line_item_blended_rate" "blended_rate" , "line_item_unblended_rate" "unblended_rate" , "pricing_public_on_demand_rate" "public_ondemand_rate" , "product_transfer_type" "data_transfer_type"
            FROM (database).(tablename)
            WHERE ((((("line_item_usage_type" LIKE '%Bytes%')
                    AND (((("line_item_usage_type" LIKE '%In%')
                    OR ("line_item_usage_type" LIKE '%Out%'))
                    OR ("line_item_usage_type" LIKE '%Regional%'))
                    AND ((NOT ("line_item_usage_type" LIKE '%Cloudfront%'))
                    AND ("line_item_usage_type" <> 'DataTransfer-In-Bytes'))))
                    AND (("product_from_location" = '')
                    OR ("product_from_location" LIKE '%(%')))
                    AND (NOT ("line_item_line_item_type" IN ('Tax', 'RIFee', 'Fee', 'Refund', 'Credit'))))
                    AND ("line_item_blended_cost" > 0.0))
                    AND (("bill_billing_period_start_date" >= ("date_trunc"('month', current_timestamp) - INTERVAL '3' MONTH))
                    AND (CAST("concat"("year", '-', "month", '-01') AS date) >= ("date_trunc"('month', current_date) - INTERVAL '3' MONTH)))
            GROUP BY  "line_item_product_code", "bill_billing_period_start_date", "line_item_usage_account_id", "bill_payer_account_id", "product_product_name", "line_item_line_item_type", "line_item_operation", "product_region", "line_item_usage_type", "product_from_location", "product_to_location", "line_item_resource_id", "line_item_blended_rate", "line_item_line_item_description", "product_transfer_type", "product_usagetype", "pricing_public_on_demand_cost", "pricing_public_on_demand_rate", "line_item_unblended_rate", "line_item_unblended_cost", "line_item_blended_cost" 


{{% /expand%}}

- {{%expand "Click here - to expand Create view in Athena using aws cli" %}}
- Copy the code below to a new file and name it **create-data-transfer-view-query.json**. Then replace the values as follows-
   

    - `<your database>.<your table>` = Your database.table

    - `<your s3 bucket>` = Your s3 bucket

    - `<your Athena Workgroup>` = Your Athena WorkGroup
   
    - Optional: Adjust the look back from '3' months to desired time-frame in QueryString

          {
                "QueryString": "CREATE OR REPLACE VIEW data_transfer_view AS SELECT DISTINCT line_item_product_code product_code, bill_billing_period_start_date billing_period, bill_payer_account_id payer_account_id, line_item_usage_account_id linked_account_id, product_product_name product_name, line_item_line_item_type charge_type, line_item_operation operation, product_region region, line_item_usage_type usage_type, product_from_location from_location , product_to_location to_location , line_item_resource_id resource_id , (sum((CASE WHEN (line_item_line_item_type = 'Usage') THEN line_item_usage_amount ELSE 0 END)) / 1024) TBs , sum((CASE WHEN (line_item_line_item_type = 'Usage') THEN line_item_usage_amount ELSE 0 END)) usage_quantity, sum(line_item_blended_cost) blended_cost , sum(line_item_unblended_cost) unblended_cost , sum(pricing_public_on_demand_cost) public_cost , line_item_blended_rate blended_rate, line_item_unblended_rate unblended_rate, pricing_public_on_demand_rate public_ondemand_rate, product_transfer_type data_transfer_type FROM <your database>.<your table> WHERE (((((line_item_usage_type LIKE '%Bytes%') AND ((((line_item_usage_type LIKE '%In%') OR (line_item_usage_type LIKE '%Out%')) OR (line_item_usage_type LIKE '%Regional%')) AND ((NOT (line_item_usage_type LIKE '%Cloudfront%')) AND (line_item_usage_type <> 'DataTransfer-In-Bytes')))) AND ((product_from_location = '') OR (product_from_location LIKE '%(%'))) AND (NOT (line_item_line_item_type IN ('Tax', 'RIFee', 'Fee', 'Refund', 'Credit')))) AND (line_item_blended_cost > 0.0)) AND ((bill_billing_period_start_date >= (date_trunc('month', current_timestamp) - INTERVAL  '3' MONTH)) AND (CAST(concat(\year\, '-', \month\, '-01') AS date) >= (date_trunc('month', current_date) - INTERVAL  '3' MONTH))) GROUP BY  line_item_product_code, bill_billing_period_start_date, line_item_usage_account_id, bill_payer_account_id, product_product_name, line_item_line_item_type, line_item_operation, product_region, line_item_usage_type, product_from_location, product_to_location, line_item_resource_id, line_item_blended_rate, line_item_line_item_description, product_transfer_type, product_usagetype, pricing_public_on_demand_cost, pricing_public_on_demand_rate, line_item_unblended_rate, line_item_unblended_cost, line_item_blended_cost",
                "QueryExecutionContext": {
                    "Database": "costmaster",
                    "Catalog": "AWSDataCatalog"
                    },
                "ResultConfiguration": {
                    "OutputLocation": "s3://<your S3 bucket>/tmp"
                    },
                "WorkGroup": "<your Athena Workgroup>"
          }

- Run the following command in a terminal window from the folder where you created **create-data-transfer-view-query.json**

        aws athena start-query-execution --cli-input-json file://create-data-transfer-view-query.json

- To check query execution status

        aws athena get-query-execution --query-execution-id <QueryExecutionId returned from previus command> --region us-east-1 
        
- Response:

![Images/get_query_dt.png](/Cost/200_Enterprise_Dashboards/Images/get_query_dt.png)

{{% /expand%}}


- Confirm the view is working, run the following Athena query and you should receive 10 rows of data:

        select * from costmaster.data_transfer_view
        limit 10;