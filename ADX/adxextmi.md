# Azure Data explorer - Managed identity - External Table

## Requirements

- Azure Account
- Azure Data Explorer
- Azure storage account
- Sample data set in parquet format

## Steps

- Load the sample data set into Azure storage account
- Setup managed identities for external table

```
.alter-merge cluster policy managed_identity ```[
    {
      "ObjectId": "system",
      "AllowedUsages": "ExternalTable"
    }
]```
```

- from documentation - https://learn.microsoft.com/en-us/azure/data-explorer/external-tables-managed-identities?tabs=system-assigned%2Cazure-storage
- now create external table

```
.create external table tpchoutput2 (registration_dttm: datetime , id: int, first_name: string, last_name: string, email: string, gender: string, ip_address: string, cc: string, country: string, birthdate: string, salary: double, title: string, comments: string )  
kind=storage  
dataformat=parquet
( 
   h@'https://storagename.blob.core.windows.net/userdata/userdata1.parquet;managed_identity=system'
)
```

- display the data

```
external_table("tpchoutput2")
| limit 10000
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/ADX/images/adxextmi-1.jpg 'RagChat')